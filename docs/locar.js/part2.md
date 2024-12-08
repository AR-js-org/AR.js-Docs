# Location-based AR.js with LocAR.js

## Part 2 - Using the GPS and Device Orientation 

Having looked at the basics of the LocAR.js API in the first tutorial, we will now look at how to use the real GPS location. Last time, if you remember, we used a "fake" location with the `LocAR.LocationBased`'s `fakeGps() `call.

We will also look at how we can use the device's orientation controls, so that the orientation sensors are tracked and objects will appear in their real-world position when the device is rotated. For example, an object directly north of the user will only appear when the device is facing north. 

### GPS tracking

Here is a revised version of the previous example which obtains your real GPS location:

```javascript
import * as THREE from 'three';
import * as LocAR from 'locar';

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(60, window.innerWidth/window.innerHeight, 0.001, 100);
const renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

window.addEventListener("resize", e => {
    renderer.setSize(window.innerWidth, window.innerHeight);
    camera.aspect = window.innerWidth / window.innerHeight;    
    camera.updateProjectionMatrix();
});
const box = new THREE.BoxGeometry(2,2,2);
const cube = new THREE.Mesh(box, new THREE.MeshBasicMaterial({ color: 0xff0000 }));

const locar = new LocAR.LocationBased(scene, camera);
const cam = new LocAR.WebcamRenderer(renderer);


locar.startGps();
locar.add(cube, -0.72, 51.0501);

renderer.setAnimationLoop(animate);


function animate() {
    cam.update();
    renderer.render(scene, camera);
}
```
Note that we only needed to make one change, we replace the `fakeGps()` call with:
```
arjs.startGps();
```
Using the Geolocation API this will make the application start listening for GPS updates. *The nice thing is we do not need to do anything else. The `LocationBased` object automatically updates the camera x and z coordinates to reflect our current GPS location.* Specifically, the GPS latitude and longitude are converted to Spherical Mercator, the sign of `z` reversed (to match the OpenGL coordinate system), and the resulting coordinates used for the camera coordinates.

### Using the device orientation controls

Having looked at obtaining our real GPS position, we will now look at how we can use the orientation controls to ensure our AR scene matches the real world as we rotate the device around. This is, in principle, quite easy: we just need to create a `LocAR.DeviceOrientationControls` object and update it in our rendering function. This object is based on the original `DeviceOrientationControls` from three.js. 

However, there is a slight problem. Unfortunately this will only work in Chrome on Android (it may also work in Chrome on iOS, this needs testing). This is due to the difficulty in obtaining absolute orientation (i.e. our orientation relative to north) using the device orientation API. This can be done on Chrome/Android using the `deviceorientationabsolute` event (and in fact, the `LocAR.DeviceOrientationControls` has been modified from the original to handle this event); it can also be done on Safari with `webkitCompassHeading` (but, due to the lack of an iDevice for testing, has not been implemented yet) but sadly it appears that support on Firefox is completely missing for now. See [this table of compatibility for absolute device orientation](https://developer.mozilla.org/en-US/docs/Web/API/Window/ondeviceorientationabsolute).

So it's recommended you use Chrome on Android for the moment. The example below shows the use of orientation tracking:

```javascript
import * as THREE from 'three';
import * as LocAR from 'locar';

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(60, window.innerWidth/window.innerHeight, 0.001, 100);
const renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

window.addEventListener("resize", e => {
    renderer.setSize(window.innerWidth, window.innerHeight);
    camera.aspect = window.innerWidth / window.innerHeight;    
    camera.updateProjectionMatrix();
});
const box = new THREE.BoxGeometry(2,2,2);

const cube = new THREE.Mesh(box, new THREE.MeshBasicMaterial({ color: 0xff0000 }));

const locar = new LocAR.LocationBased(scene, camera);
const cam = new LocAR.WebcamRenderer(renderer);

// Create the device orientation tracker
const deviceOrientationControls = new LocAR.DeviceOrientationControls(camera);

locar.startGps();
locar.add(cube, -0.72, 51.0501);

renderer.setAnimationLoop(animate);


function animate() {
    // Update the scene using the latest sensor readings
    deviceOrientationControls.update();

    cam.update();
    renderer.render(scene, camera);
}
```

Note how we create a device orientation tracker with:
```javascript
const deviceOrientationControls = new LocAR.DeviceOrientationControls(camera);
```

The device orientation tracker updates the camera, so we need to pass it in as an argument.

Also note how we update the device orientation tracker in our rendering function, so that new readings from the sensors are accounted for:

```javascript
deviceOrientationControls.update();
```

### Try it!

Try it out. As real GPS location and device orientation is used, you will need a mobile device. You should find that the red box appears in its real world position (ensure it's not too far from you, e.g. 0.001 degrees of longitude to the north) and, due to the use of orientation tracking, only appears in the field of view when you are facing its location.

### Adding four boxes to north, south, east and west

One issue on some devices with web AR is that the device's sensors (accelerometer, magnetometer) may be miscalibrated on a small number of devices. A good way of checking this is to write a simple app which displays four boxes to your north, south, east and west when a GPS location is first obtained. You can then check whether those boxes appear in their correct location. (In future we aim to produce a calibration tool to correct any miscalibration on your device).

Here is an enhanced version of the previous example, which will do this:

```javascript
import * as THREE from 'three';
import * as LocAR from 'locar';

const camera = new THREE.PerspectiveCamera(80, window.innerWidth / window.innerHeight, 0.001, 1000);

const renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

const scene = new THREE.Scene();

const locar = new LocAR.LocationBased(scene, camera);

window.addEventListener("resize", e => {
    renderer.setSize(window.innerWidth, window.innerHeight);
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
});

const cam = new LocAR.WebcamRenderer(renderer);

let firstLocation = true;

const deviceOrientationControls = new LocAR.DeviceOrientationControls(camera);

locar.on("gpsupdate", (pos, distMoved) => {
    if(firstLocation) {

        const boxProps = [{
            latDis: 0.001,
            lonDis: 0,
            colour: 0xff0000
        }, {
            latDis: -0.001,
            lonDis: 0,
            colour: 0xffff00
        }, {
            latDis: 0,
            lonDis: -0.001,
            colour: 0x00ffff
        }, {
            latDis: 0,
            lonDis: 0.001,
            colour: 0x00ff00
        }];

        const geom = new THREE.BoxGeometry(20,20,20);

        for(const boxProp of boxProps) {
            const mesh = new THREE.Mesh(
                geom, 
                new THREE.MeshBasicMaterial({color: boxProp.colour})
            );
        
            locar.add(
                mesh, 
                pos.coords.longitude + boxProp.lonDis, 
                pos.coords.latitude + boxProp.latDis
            );
        }
        
        firstLocation = false;
    }
});

locar.startGps();

renderer.setAnimationLoop(animate);

function animate() {
    cam.update();
    deviceOrientationControls.update();
    renderer.render(scene, camera);
}
```
Note how it works: when we get a location, we check whether this was the first GPS location obtained (to prevent the same boxes being added each time our GPS location changes). If it was, we add four boxes a short distance to the north (red), south (yellow), west (cyan) and east (green) of us. 

Try it out, and if your sensors are calibrated correctly, you will see a red box to your north, a yellow box to your south, a cyan (light blue) box to your west and a green box to your east. These are relative to your *initial* position so as you move, the boxes' positions relative to you will change.
