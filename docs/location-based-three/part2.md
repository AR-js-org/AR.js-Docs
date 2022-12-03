# Location-based AR.js with three.js

## Part 2 - Using the GPS and Device Orientation 

Having looked at the basics of the three.js location-based API in the first tutorial, we will now look at how to use the real GPS location. Last time, if you remember, we used a "fake" location with the `THREEx.LocationBased`'s `fakeGps() `call.

We will also look at how we can use the device's orientation controls, so that the orientation sensors are tracked and objects will appear in their real-world position when the device is rotated. For example, an object directly north of the user will only appear when the device is facing north. 

### GPS tracking

Here is a revised version of the previous example which obtains your real GPS location:

```javascript
import * as THREE from 'three';
import * as THREEx from './node_modules/@ar-js-org/ar.js/three.js/build/ar-threex-location-only.js';

function main() {
    const canvas = document.getElementById('canvas1');

    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(60, 1.33, 0.1, 10000);
    const renderer = new THREE.WebGLRenderer({canvas: canvas});

    const arjs = new THREEx.LocationBased(scene, camera);
    const cam = new THREEx.WebcamRenderer(renderer);

    const geom = new THREE.BoxGeometry(20, 20, 20);
    const mtl = new THREE.MeshBasicMaterial({color: 0xff0000});
    const box = new THREE.Mesh(geom, mtl);

    // Change this to a location 0.001 degrees of latitude north of you, so that you will face it
    arjs.add(box, -0.72, 51.051); 

    // Start the GPS
    arjs.startGps();

    requestAnimationFrame(render);

    function render() {
        if(canvas.width != canvas.clientWidth || canvas.height != canvas.clientHeight) {
            renderer.setSize(canvas.clientWidth, canvas.clientHeight, false);
            const aspect = canvas.clientWidth/canvas.clientHeight;
            camera.aspect = aspect;
            camera.updateProjectionMatrix();
        }
        cam.update();
        renderer.render(scene, camera);
        requestAnimationFrame(render);
    }
}

main();

```
Note that we only needed to make one change, we replace the `fakeGps()` call with:
```
arjs.startGps();
```
Using the Geolocation API this will make the application start listening for GPS updates. *The nice thing is we do not need to do anything else. The `LocationBased` object automatically updates the camera x and z coordinates to reflect our current GPS location.* Specifically, the GPS latitude and longitude are converted to Spherical Mercator, the sign of `z` reversed (to match the OpenGL coordinate system), and the resulting coordinates used for the camera coordinates.

### Using the device orientation controls

Having looked at obtaining our real GPS position, we will now look at how we can use the orientation controls to ensure our AR scene matches the real world as we rotate the device around. This is, in principle, quite easy: we just need to create a `THREEx.DeviceOrientationControls` object and update it in our rendering function. This object is based on the original `DeviceOrientationControls` from three.js. 

However, there is a slight problem. Unfortunately this will only work in Chrome on Android (it may also work in Chrome on iOS, this needs testing). This is due to the difficulty in obtaining absolute orientation (i.e. our orientation relative to north) using the device orientation API. This can be done on Chrome/Android using the `deviceorientationabsolute` event (and in fact, the `THREEx.DeviceOrientationControls` has been modified from the original to handle this event); it can also be done on Safari with `webkitCompassHeading` (but, due to the lack of an iDevice for testing, has not been implemented yet) but sadly it appears that support on Firefox is completely missing for now. See [this table of compatibility for absolute device orientation](https://developer.mozilla.org/en-US/docs/Web/API/Window/ondeviceorientationabsolute).

So it's recommended you use Chrome on Android for the moment. The example below shows the use of orientation tracking:

```javascript
const canvas = document.getElementById('canvas1');

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(60, 1.33, 0.1, 10000);
const renderer = new THREE.WebGLRenderer({canvas: canvas});

const arjs = new THREEx.LocationBased(scene, camera);
const cam = new THREEx.WebcamRenderer(renderer);

const geom = new THREE.BoxGeometry(20, 20, 20);
const mtl = new THREE.MeshBasicMaterial({color: 0xff0000});
const box = new THREE.Mesh(geom, mtl);

// Create the device orientation tracker
const deviceOrientationControls = new THREEx.DeviceOrientationControls(camera);

// Change this to a location close to you (e.g. 0.001 degrees of latitude north of you)
arjs.add(box, -0.72, 51.051); 

arjs.startGps();

requestAnimationFrame(render);

function render() {
    if(canvas.width != canvas.clientWidth || canvas.height != canvas.clientHeight) {
        renderer.setSize(canvas.clientWidth, canvas.clientHeight, false);
        const aspect = canvas.clientWidth/canvas.clientHeight;
        camera.aspect = aspect;
        camera.updateProjectionMatrix();
    }

    // Update the scene using the latest sensor readings
    deviceOrientationControls.update();

    cam.update();
    renderer.render(scene, camera);
    requestAnimationFrame(render);
}
```

Note how we create a device orientation tracker with:
```javascript
const deviceOrientationControls = new THREEx.DeviceOrientationControls(camera);
```

The device orientation tracker updates the camera, so we need to pass it in as an argument.

Also note how we update the device orientation tracker in our rendering function, so that new readings from the sensors are accounted for:

```javascript
deviceOrientationControls.update();
```

### Try it!

Try it out. As real GPS location and device orientation is used, you will need a mobile device. You should find that the red box appears in its real world position (ensure it's not too far from you, e.g. 0.001 degrees of longitude to the north) and, due to the use of orientation tracking, only appears in the field of view when you are facing its location.

