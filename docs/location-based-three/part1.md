# Location-based AR.js with three.js

## Part 1 - Hello World!


The first part of this tutorial will show you how to create a "hello world" application using the pure three.js API for location-based ar.js. It is assumed you are aware of basic three.js concepts, such as the scene, renderer and camera as well as geometries, materials and meshes. This example will set your location to a "fake" GPS location and add a box a short distance away.

Let's start with the HTML:

```html
<!DOCTYPE html>
<html>
<head>
<title>Location-based AR.js with three.js</title>
<script type='module' src='dist/bundle.js'></script>
</head>
<body>
<canvas id='canvas1' style='background-color: black; width: 100%; height: 100%'></canvas>
</body>
</html>
```

This example assumes that you have installed AR.js via `npm` and used Webpack to build the application, as described on the [index page for the tutorial](index.md). We link in the built bundle of our own code plus the three.js and AR.js dependencies. Here is our own code: save this as `index.js`.

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
    arjs.add(box, -0.72, 51.051); 


    arjs.fakeGps(-0.72, 51.05);

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

Much of this should be familiar to you from basic three.js examples, and is written in the same style as the [manual](https://threejs.org/manual). As normal, we create a `THREE.Scene`, a `THREE.PerspectiveCamera` and a `THREE.WebGLRenderer` using our canvas. 

What comes next though is new, and specific to AR.js:

```javascript
const arjs = new THREEx.LocationBased(scene, camera);
const cam = new THREEx.WebcamRenderer(renderer);
```

We use two new objects, both part of the AR.js API. Firstly `THREEx.LocationBased` is the overall AR.js "manager" object and secondly `THREEx.WebcamRenderer` is responsible for rendering the camera feed. We need to supply our scene and camera as arguments to `THREEx.LocationBased` and our renderer as an argument to `THREEx.WebcamRenderer`.

The `THREEx.WebcamRenderer` will create a `video` element to capture the webcam. Alternatively, if you have a `video` element already set up in your HTML, you can pass its CSS selector into the `WebcamRenderer` as an optional argument. For example:

```javascript
const cam = new THREEx.WebcamRenderer(renderer, '#video1');
```

Next, using standard three.js code, we set up a mesh using a box geometry and red material (i.e. a red box). However, this is where it then gets interesting:

```javascript
arjs.add(box, -0.72, 51.051); 
```

Rather than setting the box's `position` as we would normally do in standard three.js, we add it to a specific **real-world location** defined by longitude and latitude. The `add()` method of `THREEx.LocationBased` allows us to do that.

Having positioned our box in a specific real-world location, we now need to place **ourselves** (i.e. the camera) at a given real-world location We can do this with `THREEx.LocationBased`s `fakeGps()` method, which takes longitude and latitude as parameters:

```javascript
arjs.fakeGps(-0.72, 51.05);
```

This plaves us just to the south of the red box. By default, we face north, so the red box will appear in front of us.

The remaining code is the standard three.js code for rendering each frame, and dealing with potential screen resizes. However note this code within the rendering function:

```javascript
cam.update();
```

This API call will render the latest camera frame.

### Try it!

Try it on either a desktop machine or an Android device running Chrome. On a mobile device or desktop you should see the feed from the webcam, and a red box just in front of you. Note that the mobile device will not yet respond to changes in orientation: we will add that next time. For this reason you *must ensure the box is to your north* as the default view is to face north.

### Faking rotation on a desktop machine

If you do not have a suitable mobile device, you can simulate rotation with the mouse. The code below will do this (add to your main block of code, just before the rendering function):

```javascript
const rotationStep = THREE.Math.degToRad(2);

let mousedown = false, lastX =0;

window.addEventListener("mousedown", e=> {
    mousedown = true;
});

window.addEventListener("mouseup", e=> {
    mousedown = false;
});

window.addEventListener("mousemove", e=> {
    if(!mousedown) return;
    if(e.clientX < lastX) {
        camera.rotation.y -= rotationStep;
        if(camera.rotation.y < 0) {
            camera.rotation.y += 2 * Math.PI;
        }
    } else if (e.clientX > lastX) {
        camera.rotation.y += rotationStep;
        if(camera.rotation.y > 2 * Math.PI) {
            camera.rotation.y -= 2 * Math.PI;
        }
    }
    lastX = e.clientX;
});
```

What does this do? Using mouse events, it detects the direction of movement of the mouse when it's pressed down, and in doing so, determines whether to rotate the camera clockwise or anticlockwise. It does this using the `clientX` property of the event object, which contains the mouse X position. This is compared to the previous value of `e.clientX` and from this, we can determine whether we moved the mouse to the left or to the right, and rotate accordingly.

We move the camera by the amount specified in `rotationStep` and ensure that the camera rotation is always within the range 0 to 2PI radians (i.e. 360 degrees).

