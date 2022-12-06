# AR.js A-Frame Location-Based Tutorial - Develop a Simple Points of Interest App 

## Introduction

This tutorial (**updated for AR.js 3.4**) aims to take you from a basic location-based AR.js example all the way to a working, simple points of interest app. We will start with an HTML-only example and gradually add JavaScript to make our app more sophisticated.

It is expected that you have some basic [A-Frame](https://aframe.io) experience.

**Do note that this code will not work on Firefox on a mobile device due to limitations of the device orientation API; absolute orientation cannot be obtained. Chrome on Android is recommended.**

## Basic example 

We will start with a basic example, using pure HTML, to display a box close
to your location. This example is identical to the location-based example on the
[index page](../index.md).

```html
<!DOCTYPE html>
<html>
<head>
<title>AR.js A-Frame Location-based</title>
<script src="https://aframe.io/releases/1.0.4/aframe.min.js"></script>
<script type='text/javascript' src='https://raw.githack.com/AR-js-org/AR.js/master/three.js/build/ar-threex-location-only.js'></script>
<script type='text/javascript' src='https://raw.githack.com/AR-js-org/AR.js/master/aframe/build/aframe-ar.js'></script>
</head>
<body>
<a-scene vr-mode-ui='enabled: false' arjs='sourceType: webcam; videoTexture: true; debugUIEnabled: false' renderer='antialias: true; alpha: true'>
    <a-camera gps-new-camera='gpsMinDistance: 5'></a-camera>
    <a-entity material='color: red' geometry='primitive: box' gps-new-entity-place="latitude: <your-lat>; longitude: <your-loon>" scale="10 10 10"></a-entity>
</a-scene>
</body>
</html>
```
Upload this to a server with HTTPS, or run locally on `localhost`. Make
sure you replace `your-lat` and `your-lon` with values close to your
actual position (to see the box clearly, I would recommend an offset of 
around 0.001 degrees in any direction for both the latitude and longitude).

### How does this work?

- The `arjs` component of our `a-scene` initialises AR.js. Note the properties
we are setting: we set the `sourceType` to `webcam` for obvious reasons
but also set `videoTexture` to true. This is vital in an outdoor 
location-based AR app as it allows distant augmented content - such at the
peaks we are going to eventually visualise - to be seen. (It does this by
using a three.js texture for the camera feed which can be easily combined with
our augmented content).

- Note the `gps-new-camera` component on our `a-camera`. This is the 
AR.js component which automatically
converts latitudes and longitudes into 3D world coordinates, allowing us to
use latitude and longitude, rather than world coordinates, when adding places.
Note that we are using `gps-new-camera`, not `gps-camera`. The
`gps-new-camera` component includes some bugfixes and makes it easy for us to 
work with arbitrary
geographical data provided by a server, as internally it uses the
Spherical Mercator projection to represent the augmented content's world
coordinates. Spherical Mercator units are commonly used to represent 
mapping data and are almost (but not quite) equivalent to metres. Away from the polar regions, though, it's good enough to use for AR.

- We then create an `a-box` primitive. This is the augmented content that we
want to display. Ordinarily, in A-Frame, you would give this a `position` in
world coordinates. However, AR.js, and specifically the `gps-new-entity-place` component, allows us to position it using latitude and longitude. 
We can position any A-Frame entity at a given latitude and longitude using
`gps-projected-entity-place`.


### Things to try

- Change the `a-box` to some other kind of A-Frame primitive, such as an
`a-sphere` or `a-cylinder`. Does it still work?

- Try adding multiple objects with different colours at different locations.

- Try adding a text primitive at a nearby latitude and longitude. 
You will need to use the A-Frame `look-at` component to ensure the text always 
faces the camera.

- Try giving your objects an elevation. This can be done by setting the
`y` coordinate of the `position` property of each object to a given height 
(in metres) and setting the `x` and `z` coordinates to 0.

- Having done that, try giving the *camera* an elevation by similarly setting its `position` property, and look at the effect this has on where the objects appear.


## Introducing JavaScript with AR.js

Much of the power of A-Frame, and AR.js, comes from adding scripting to your
basic applications. It is assumed that you already know the basics of how
to create components in A-Frame. We will start with a very basic example, which simply retrieves your current GPS location and adds a red box immediately to the north. Create this JavaScript, `basic.js`, and link it to the HTML example shown above. (Remove the hard-coded red box from the HTML first).

```javascript
window.onload = () => {
    let testEntityAdded = false;

    const el = document.querySelector("[gps-new-camera]");

    el.addEventListener("gps-camera-update-position", e => {
        if(!testEntityAdded) {
            alert(`Got first GPS position: lon ${e.detail.position.longitude} lat ${e.detail.position.latitude}`);
            // Add a box to the north of the initial GPS position
            const entity = document.createElement("a-box");
            entity.setAttribute("scale", {
                x: 20, 
                y: 20,
                z: 20
            });
            entity.setAttribute('material', { color: 'red' } );
            entity.setAttribute('gps-new-entity-place', {
                latitude: e.detail.position.latitude + 0.001,
                longitude: e.detail.position.longitude
            });
            document.querySelector("a-scene").appendChild(entity);
        }
        testEntityAdded = true;
    });
};
```

How is this working?

- we set up an `onload` function to run when the page loads. With A-Frame, we can only use entities once they have been loaded into the DOM, so we must delay the execution of the code until the page loads.
- Using the normal DOM API, we use `document.querySelector()` to obtain the entity with the `gps-new-camera` component attached to it (which will be your `<a-camera>`)
- we then handle the `gps-camera-update-position` **event**. This event is emitted by the camera entity when we receive a new GPS location. This allows us to write code which runs every time we get a new GPS position, such as downloading new POI data from a server. We can retrieve the new location via the `e.detail.position` object, which has `longitude` and `latitude` properties.
- In this example, we check that we have not already added our entity (via the `testEntityAdded` boolean), display the location to the user, and then create a new `<a-box>` entity dynamically, and specify its scale and colour using standard DOM/A-Frame techniques.
- We then **dynamically** add a `gps-new-entity-place` component to the entity, with the latitude set to the GPS latitude plus 0.001 degrees (so it will appear a short distance to the north) and the longitude set to the current GPS longitude.
- Finally we add the entity to the scene using the standard DOM `appendChild()` method.

### Things to try

Add three more entities to the scene, close to the original GPS position

- a yellow sphere 0.001 degrees to the east;
- an orange cylinder 0.001 degrees to the south;
- a magenta cone 0.001 degrees to the west.

## Connecting to a web server

We will now enhance the example to *download data from a web server*. The server used will be the Hikar server, used by the [Hikar](https://hikar.org) project:
```
https://hikar.org/webapp/map?bbox=west,south,east,north&layers=poi&outProj=4326
```
This provides [OpenStreetMap](https://openstreetmap.org) data for Europe and Turkey (apologies, other parts of the world are not covered due to server constraints). Note how we specify the bounding box with the `bbox` parameter.

```javascript
window.onload = () => {
    let downloaded = false;

    const el = document.querySelector("[gps-new-camera]");

    el.addEventListener("gps-camera-update-position", async(e) => {
        if(!downloaded) {
            const west = e.detail.position.longitude - 0.01,
                  east = e.detail.position.longitude + 0.01,
                  south = e.detail.position.latitude - 0.01;
                  north = e.detail.position.latitude + 0.01;
            const response = await fetch(`https://hikar.org/webapp/map?bbox=${west},${south},${east},${north}&layers=poi&outProj=4326`);
            const pois = await response.json();
            pois.features.forEach ( feature => {
                const entity = document.createElement("a-box");
                entity.setAttribute("scale", {
                    x: 20, 
                    y: 20,
                    z: 20
                });
                entity.setAttribute('material', { color: 'red' } );
                entity.setAttribute('gps-new-entity-place', {
                    latitude: feature.geometry.coordinates[1],
                    longitude: feature.geometry.coordinates[0]
                }); 
                document.querySelector("a-scene").appendChild(entity);
            });
        }
        downloaded = true;
    });
};
```
Much of the logic is similar to the previous example, but note that we now send a request to the web server via the `fetch` API, sending a bounding box surrounding the current position. The server sends back [GeoJSON](https://geojson.org). GeoJSON contains a `features` array containing each point of interest, and each feature includes a `geometry` object containing the latitude and longitude witthin a two-member `coordinates` array. So we loop through each feature, dynamically create an entity (as in the previous example) from the current feature, use the latitude and longitude from the GeoJSON to create the `gps-new-entity-place` component, and add it to the scene.

### Things to try

Try requesting the Hikar URL directly in your browser, supplying a bounding box representing an area you are familiar with, and explore the format used for points of interest of different types. Each GeoJSON feature object has a `properties` object containing properties describing the point of interest. The `amenity` property is commonly used: this describes the type of amenity (such as restaurant, cafe, pub, etc).

Try colouring the boxes differently depending on point of interest type (e.g. restaurants, cafes, pubs, etc).

## Adding text labels

The next example shows how you can add text labels to your POIs.

```javascript
window.onload = () => {
    let downloaded = false;

    const el = document.querySelector("[gps-new-camera]");

    el.addEventListener("gps-camera-update-position", async(e) => {
        if(!downloaded) {
            const west = e.detail.position.longitude - 0.05,
                  east = e.detail.position.longitude + 0.05,
                  south = e.detail.position.latitude - 0.05;
                  north = e.detail.position.latitude + 0.05;
            console.log(`${west} ${south} ${east} ${north}`);
            const response = await fetch(`https://hikar.org/webapp/map?bbox=${west},${south},${east},${north}&layers=poi&outProj=4326`);
            const pois = await response.json();
            pois.features.forEach ( feature => {
                const compoundEntity = document.createElement("a-entity");
                compoundEntity.setAttribute('gps-new-entity-place', {
                    latitude: feature.geometry.coordinates[1],
                    longitude: feature.geometry.coordinates[0]
                });
                const box = document.createElement("a-box");
                box.setAttribute("scale", {
                    x: 20,
                    y: 20,
                    z: 20
                });
                box.setAttribute('material', { color: 'red' } );
                box.setAttribute("position", {
                    x : 0,
                    y : 20,
                    z: 0
                } );
                const text = document.createElement("a-text");
                const textScale = 100;
                text.setAttribute("look-at", "[gps-new-camera]");
                text.setAttribute("scale", {
                    x: textScale,
                    y: textScale,
                    z: textScale
                });
                text.setAttribute("value", feature.properties.name);
                text.setAttribute("align", "center");
                compoundEntity.appendChild(box);
                compoundEntity.appendChild(text);
                document.querySelector("a-scene").appendChild(compoundEntity);
            });
        }
        downloaded = true;
    });
};
```

How is this working?

- We now create a *compound entity*. In A-Frame, a compound entity is an entity which has other entities as children. Here, we will create a compound entity, position it at the POI's latitude and longitude, and add the box, and a new text entity containing the POI name, to it.
- We create the box as before, and set its `y` coordinate to 20. This is relative to its parent, i.e. the compound entity. The compound entity is already positioned at the correct latitude and longitude, so we will position the box 20 metres above that position.
- We then create a text entity, scale it appropriately, and set its `value` attribute to the name of the feature from the GeoJSON. Note the use of the `look-at` component. This makes a given A-Frame entity look at another. Here we want the text to look at the camera (i.e. the entity with a `gps-new-camera` property), so we always see it. The `look-at` component is a third-party component and must be added to your HTML as follows:
```html
<script src="https://unpkg.com/aframe-look-at-component@0.8.0/dist/aframe-look-at-component.min.js"></script>
```
- We then append the box and text to the compound entity, and append our compound entity to the scene.

### Things to try

- Try filtering out POIs with no name, so that only those with a name are displayed. Those without a name should not be displayed, not even as a box. 
- Try implementing logic to re-download POIs if the user moves 0.05 degrees of either latitude or longitude from the previous download position.
- Each POI in the GeoJSON includes an `osm_id` property which is a unique OpenStreetMap ID for that POI. Using the `osm_id`, implement logic so that a POI is not re-added to the scene if it is already present. (This may happen if you move 0.05 degrees but return to an area you have already visited).
