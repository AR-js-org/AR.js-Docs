# Location-based AR.js with three.js

## Part 3 - Connecting to a web API 

Having looked at how to use the three.js location-based API, we will now consider an example which connects to a web API providing points of interest. This example does not actually introduce any new AR.js concepts, but shows you how you can work with a web API.

```javascript
const canvas = document.getElementById('canvas1');

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(60, 1.33, 0.1, 10000);
const renderer = new THREE.WebGLRenderer({canvas: canvas});

const arjs = new THREEx.LocationBased(scene, camera);
const cam = new THREEx.WebcamRenderer(renderer);

const geom = new THREE.BoxGeometry(20, 20, 20);
const mtl = new THREE.MeshBasicMaterial({color: 0xff0000});

const deviceOrientationControls = new THREEx.DeviceOrientationControls(camera);

let fetched = false;

// Handle the "gpsupdate" event on the LocationBased object
// This triggers when a GPS update (from the Geolocation API) occurs
// 'pos' is the position object from the Geolocation API.

arjs.on("gpsupdate", async(pos) => {
    if(!fetched) {
        const response = await fetch(`https://hikar.org/webapp/map?bbox=${pos.coords.longitude-0.01},${pos.coords.latitude-0.01},${pos.coords.longitude+0.01},${pos.coords.latitude+0.01}&layers=poi&outProj=4326`);

        const geojson = await response.json();

        geojson.features.forEach ( feature => {
            const box = new THREE.Mesh(geom, mtl);
            arjs.add(box, feature.geometry.coordinates[0], feature.geometry.coordinates[1]);            
        });
        
        fetched = true;
    }
});

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

How is this working? The key thing is we **handle the `gpsupdate` event** emitted by the `LocationBased` object when a GPS update occurs. This is specifically emitted when the inbuilt Geolocation API receives a GPS update, and allows us to trigger certain code.

Here, we trigger a download from a web API when we get the update. Note that the `gpsupdate` event handler receives the standard position object of the Geolocation API, so that, for example, its `coords` property contains the longitude and latitude. We then download data in a 0.02 x 0.02 degree box centred on our current location from the API at https://hikar.org. This provides [OpenStreetMap](https://openstreetmap.org) POI data, but only for Europe and Turkey due to server capacity constraints. The data is provided as [GeoJSON](https://geojson.org).

So having received the data, we simply loop through it and create one `THREE.Mesh` for each POI, adding it at the appropriate location (accessible via the `coordinates` of the `geometry` of each GeoJSON object).

Note the boolean variable `fetched` which is set to true as soon as we have fetched the data. This prevents data being continuously downloaded from the server every time we get a position update, as it's set to `false` as soon as data has been downloaded. In a real application you could implement code to download data by tile, so that new data is downloaded whenever you move into a new tile.
