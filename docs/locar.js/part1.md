# Location-based AR.js with LocAR.js

## Part 1 - Hello World!


The first part of this tutorial will show you how to create a "hello world" application using LocAR.js. It is assumed you are aware of basic three.js concepts, such as the scene, renderer and camera as well as geometries, materials and meshes. This example will set your location to a "fake" GPS location and add a box a short distance away.

Let's start with the HTML, which is very simple:

```html
<!DOCTYPE html>
<html>
<head>
<title>LocAR.js - Hello World</title>
<script type='module' src='src/main.js'></script>
</head>
<body>
</body>
</html>
```

This example assumes that you have installed LocAR.js via `npm` and are using Vite in dev mode to run the application, as described on the [index page for the tutorial](index.md). We link in our JavaScript source as an ES6 module from `src/main.js`, so this is where you should save your code, as `main.js` inside the `src` directory. Here is the `main.js` code:

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


locar.fakeGps(-0.72, 51.05);
locar.add(cube, -0.72, 51.0501);

renderer.setAnimationLoop(animate);


function animate() {
    cam.update();
    renderer.render(scene, camera);
}

```

Much of this is using standard three.js setup code as described in the [manual](https://threejs.org/docs/index.html#manual/en/introduction/Creating-a-scene); if you do not understand basic three.js concepts such as the scene, camera, and renderer, as well as geometries and meshes, you should read the three.js manual first. 

As normal, we create a `THREE.Scene`, a `THREE.PerspectiveCamera` and a `THREE.WebGLRenderer` using our canvas. We also handle resizing the window and handle window resize. We then create a box geometry and a mesh using that box geometry.

What comes next though is new, and specific to AR.js:

```javascript
const locar = new LocAR.LocationBased(scene, camera);
const cam = new LocAR.WebcamRenderer(renderer);
```

We use two new objects, both part of the LocAR.js API. Firstly `LocAR.LocationBased` is the overall AR.js "manager" object and secondly `LocAR.WebcamRenderer` is responsible for rendering the live camera feed. We need to supply our scene and camera as arguments to `LocAR.LocationBased` and our renderer as an argument to `LocAR.WebcamRenderer`.

The `LocAR.WebcamRenderer` will, internally, create a `video` element to capture the webcam. Alternatively, if you have a `video` element already set up in your HTML, you can pass its CSS selector into the `WebcamRenderer` as an optional argument. For example:

```javascript
const cam = new LocAR.WebcamRenderer(renderer, '#video1');
```

Next we add our box mesh to LocAR. This next line is interesting: 

```javascript
locar.add(box, -0.72, 51.051); 
```

Rather than setting the box's `position` as we would normally do in standard three.js, we add it to a specific **real-world location** defined by longitude and latitude. The `add()` method of `LocAR.LocationBased` allows us to do that.

Having positioned our box in a specific real-world location, we now need to place **ourselves** (i.e. the camera) at a given real-world location We can do this with `LocAR.LocationBased`s `fakeGps()` method, which takes longitude and latitude as parameters:

```javascript
arjs.fakeGps(-0.72, 51.05);
```

This places us just to the south of the red box. By default, we face north, so the red box will appear in front of us.

The remaining code is the standard three.js code for defining a rendering function and setting it as the animation loop. However note this code within the rendering function:

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

