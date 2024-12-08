# Location-based AR.js with LocAR.js

## Part 3 - Connecting to a web API 

Having looked at how to use the LocAR.js API, we will now consider an example which connects to a web API providing points of interest. This example does not actually introduce any new AR.js concepts, but shows you how you can work with a web API.

```javascript
import * as THREE from 'three';
import * as LocAR from 'locar';

const camera = new THREE.PerspectiveCamera(80, window.innerWidth/window.innerHeight, 0.001, 1000);
const renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
const scene = new THREE.Scene();


document.body.appendChild(renderer.domElement);


window.addEventListener("resize", e => {
    renderer.setSize(window.innerWidth, window.innerHeight);
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
});

const locar = new LocAR.LocationBased(scene, camera);

const deviceControls = new LocAR.DeviceOrientationControls(camera);

const cam = new LocAR.WebcamRenderer(renderer);


let firstPosition = true;

const indexedObjects = { };

const cube = new THREE.BoxGeometry(20, 20, 20);

const clickHandler = new LocAR.ClickHandler(renderer);

locar.on("gpsupdate", async(pos, distMoved) => {
    
    if(firstPosition || distMoved > 100) {

        const response = await fetch(`https://hikar.org/webapp/map?bbox=${pos.coords.longitude-0.02},${pos.coords.latitude-0.02},${pos.coords.longitude+0.02},${pos.coords.latitude+0.02}&layers=poi&outProj=4326`);
        const pois = await response.json();

        pois.features.forEach ( poi => {
            if(!indexedObjects[poi.properties.osm_id]) {
                const mesh = new THREE.Mesh(
                    cube,
                    new THREE.MeshBasicMaterial({color: 0xff0000})
                );                

                locar.add(
                    mesh, 
                    poi.geometry.coordinates[0], 
                    poi.geometry.coordinates[1]
                );
                indexedObjects[poi.properties.osm_id] = mesh;
            }
        });
        firstPosition = false;
    }

});
locar.startGps();

renderer.setAnimationLoop(animate);

function animate() {
    cam.update();
    deviceControls.update();
    renderer.render(scene, camera);
}

```

How is this working? The key thing is we **handle the `gpsupdate` event** emitted by the `LocationBased` object when a GPS update occurs. This is specifically emitted when the inbuilt Geolocation API receives a GPS update, and allows us to trigger certain code.

Here, we trigger a download from a web API when we get the update. Note that the `gpsupdate` event handler receives the standard position object of the Geolocation API, so that, for example, its `coords` property contains the longitude and latitude. We then download data in a 0.02 x 0.02 degree box centred on our current location from the API at https://hikar.org. This provides [OpenStreetMap](https://openstreetmap.org) POI data, but only for Europe and Turkey due to server capacity constraints. The data is provided as [GeoJSON](https://geojson.org).

So having received the data, we simply loop through it and create one `THREE.Mesh` for each POI, adding it at the appropriate location (accessible via the `coordinates` of the `geometry` of each GeoJSON object).

Note the boolean variable `firstPosition` which is set to false as soon as we have fetched the data. This prevents data being continuously downloaded from the server every time we get a position update, as it's set to `false` as soon as data has been downloaded. In a real application you could implement code to download data by tile, so that new data is downloaded whenever you move into a new tile.

### Detecting clicks with raycasting

We can add the facility to detect clicks on our AR objects by making use of the three.js *raycaster*. This works by sending a line (ray) from a particular point in a certain direction (here, from the camera into the scene) and detecting intersections with objects. LocAR.js provides the `ClickHandler` class, a wrapper round the inbuilt three.js raycaster, to simplify the code. We create a `ClickHandler` object, passing in the renderer as an argument:

```javascript
const clickHandler = new LocAR.ClickHandler(renderer);
```

Before we add our raycasting code, we need to modify our code to add objects to the scene so that the object properties (name, etc) are specified:
```javascript
locar.add(
    mesh, 
    poi.geometry.coordinates[0], 
    poi.geometry.coordinates[1],     
    0, 
    poi.properties
);
```
Note how we specify two additional arguments when adding the object: the elevation (0; a future tutorial will show you how to add elevation from a Digital Elevation Model) and the POI properties from the GeoJSON. These will be used to display the object name to the user when we do our raycasting below.

We perform the raycasting with the `raycast()` method within our animate function. Here is an example:

```javascript
function animate() {
    cam.update();
    deviceControls.update();
    const objects = clickHandler.raycast(camera, scene);
    if(objects.length) {
        alert(`This is ${objects[0].object.properties.name}`);
    }
    renderer.render(scene, camera);
}
```
Note how `raycast()` takes the point to raycast from (the camera) and the object to raycast into (the scene). It returns an array of objects intersected by the ray. We are likely to be interested in only the first (the closest to the camera) so we obtain that using index 0. The object returned has an `object` property representing the AR object we found. The properties we specified when we added the AR object can be obtained via the `properties` property of this object. From these, we obtain the name of the object and display it as an alert box.
