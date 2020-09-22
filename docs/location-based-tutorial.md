# AR.js Location-Based Tutorial - Develop a Simple Peakfinder App 

## Introduction

This tutorial aims to take you from a basic location-based AR.js example all
the way to a working, simple peak-finder app. We will start with an HTML-only
example and gradually add JavaScript to make our app more sophisticated.

It is expected that you have some basic [A-Frame](https://aframe.io) experience.

## Basic example 

We will start with a basic example, using pure HTML, to display a box close
to your location. This example is similar to the location-based example on the
[index page](index.md).

```html
<html>
    <head>
        <title>AR.js Basic Projected Camera Example</title>
        <script src="https://aframe.io/releases/1.0.4/aframe.min.js"></script>
        <script src="https://raw.githack.com/AR-js-org/AR.js/master/aframe/build/aframe-ar-nft.js"></script>
        <!-- Look-at component. We don't need this now, but we will later. -->
        <script src="https://unpkg.com/aframe-look-at-component@0.8.0/dist/aframe-look-at-component.min.js"></script>
    </head>
    <body>
    <a-scene
        vr-mode-ui="enabled: false"
        arjs='sourceType: webcam; videoTexture: true; debugUIEnabled: false;'
        renderer='antialias: true; alpha: true'>
            <a-camera gps-projected-camera rotation-reader></a-camera>
            <a-box gps-projected-entity-place='latitude: your-lat; longitude: your-lon' material='color: red' scale='10 10 10'></a-box>
        </a-scene>
    </body>
</html>
```

Upload this to a server with HTTPS, or run locally on `localhost`. Make
sure you replace `your-lat` and `your-lon` with values close to your
actual position (to see the box clearly, I would recommend an offset of 
around 0.0004 degrees in any direction for both the latitude and longitude).

### How does this work?

- This is simplar to the example on the [index page](index.md). 

- The `arjs` component of our `a-scene` initialises AR.js. Note the properties
we are setting: we set the `sourceType` to `webcam` for obvious reasons
but also set `videoTexture` to true. This is vital in an outdoor 
location-based AR app as it allows distant augmented content - such at the
peaks we are going to eventually visualise - to be seen. (It does this by
using a three.js texture for the camera feed which can be easily combined with
our augmented content).

- Note the `gps-projected-camera` component on our `a-camera`. This is the 
AR.js component which automatically
converts latitudes and longitudes into 3D world coordinates, allowing us to
use latitude and longitude, rather than world coordinates, when adding places.
Note that we are using `gps-projected-camera`, not `gps-camera`. The
`gps-projected-camera` component makes it easy for us to work with arbitrary
geographical data provided by a server, as internally it uses the
Spherical Mercator projection to represent the augmented content's world
coordinates. Spherical Mercator units are commonly used to represent 
mapping data and are almost (but not quite) equivalent to metres. Away from the polar regions, though, it's good enough to use for AR.

- We then create an `a-box` primitive. This is the augmented content that we
want to display. Ordinarily, in A-Frame, you would give this a `position` in
world coordinates. However, AR.js, and specifically the `gps-projected-entity-place` component, allows us to position it using latitude and longitude. 
We can position any A-Frame entity at a given latitude and longitude using
`gps-projected-entity-place`.


### Things to try

- Change the `a-box` to some other kind of A-Frame primitive, such as an
`a-sphere` or `a-cylinder`. Does it still work?

- Try adding multiple objects with different colours at different locations.

- Try adding a text primitive at a nearby latitude and longitude. Use the
location-based example on the [index.page](index.md) to help you. You will
need to use the A-Frame `look-at` component to ensure the text always 
faces the camera.

- Try giving your objects an elevation. This can be done by setting the
`y` coordinate of the `position` property of each object to a given height 
(in metres) and setting the `x` and `z` coordinates to 0.

- Having done that, try giving the *camera* an elevation by similarly setting its `position` property, and look at the effect this has on where the objects appear.


## Introducing JavaScript with AR.js

Much of the power of A-Frame, and AR.js, comes from adding scripting to your
basic applications. It is assumed that you already know the basics of how
to create components in A-Frame, but let's start with a refresher. Create
this file, `peakfinder.js`:

```javascript
AFRAME.registerComponent('peakfinder', {
    init: function() {
        alert('Peakfinder component initialising!');
    }
});
```

and link to it from your HTML by adding, in the head section after the
A-Frame and AR.js scripts:

```html
<script type='text/javascript' src='peakfinder.js'></script>
```

Also add a new entity to your scene and add this component to it:

```html
<a-entity peakfinder></a-entity>
```

Load your page again. You should see the alert box come up. Remember from
basic A-Frame that to create a component, you need to register it as in the
example above. Remember also that the `init()` function runs when the component
is first initialised, and you add components to entities by making them
attributes of the `<a-entity>` tag.

### Connecting to a web API

Now we've got a basic component set up, we are going to make it do something:
connect to a web API to retrieve the locations of nearby peaks - with our
eventual aim of making a peakfinder app. There are various APIs which can
be used, but we will use one which is based on [OpenStreetMap](https://openstreetmap.org) data. This API is present on the [Hikar](https://hikar.org) web server and delivers the peak data as GeoJSON, but only covers Europe, Turkey and Washington State, USA, due to server capacity constraints. However, please feel free to research alternative APIs which cover other parts of the world and adjust the code accordingly. Also please feel free to contact me(@nickw1 on GitHub) if you want to add your area of the world (please do not ask, however, for large, heavily-populated countries in their entirety; to give you an idea of the size I could accommodate, the Washington State coverage was the result of a request). Also, note that the API is open source so again contact me for guidance on how you can set it up on your own server to cover your area of the world.

Modify your code as follows:

```javascript
AFRAME.registerComponent('peakfinder', {
    init: function() {
        this.loaded = false;
        window.addEventListener('gps-camera-update-position', e => {
            if(this.loaded === false) {
                this._loadPeaks(e.detail.position.longitude, e.detail.position.latitude);
                this.loaded = true;
            }
        });
    }
});
```

What is this doing? We are listening for the `gps-camera-update-position` event. This is emitted by `gps-projected-camera` whenever our GPS location changes. The detail of the event contains a `position` object with `longitude` and `latitude` properties which contain our current position. So when we have obtained our position, we call an (as yet unwritten) `_loadPeaks()` method which will actually download the locations of the peaks from a web API.

Note also the `loaded` boolean. This is used to prevent the peaks being loaded *every time the position changes*, which will clearly result in unnecessary network communication. For now, we just download the peaks once, when the application is initialised.

So next, we will write the `_loadPeaks()` method. Add this as a new method to your component:

```javascript
 _loadPeaks: function(longitude, latitude) {
        const scale = 2000;
        fetch(`https://www.hikar.org/fm/ws/bsvr.php?bbox=${longitude-0.1},${latitude-0.1},${longitude+0.1},${latitude+0.1}&outProj=4326&format=json&poi=natural`
            )
        .then ( response => response.json() )
        .then ( json => {
            json.features.filter ( f => f.properties.natural == 'peak' )
                .forEach ( peak => {
                    const entity = document.createElement('a-text');
                    entity.setAttribute('look-at', '[gps-projected-camera]');
                    entity.setAttribute('value', peak.properties.name);
                    entity.setAttribute('scale', {
                        x: scale,
                        y: scale,
                        z: scale
                    });
                    entity.setAttribute('gps-projected-entity-place', {
                        latitude: peak.geometry.coordinates[1],
                        longitude: peak.geometry.coordinates[0]
                    });
                    this.el.appendChild(entity);
                });
        });
    }
```

How is this working?

- We first send a standard AJAX request to our API, using `fetch`, and using a bounding box of width and height 0.2 degrees centered on our current location.
- We obtain JSON from the response using the standard `fetch` promise-based approach.
- Once we have our JSON (specifically, GeoJSON) as an array of features, we filter them to select only peaks.
- For each peak, we:
    - Create an `a-text` primitive and set it to `look-at` the `gps-projected-camera` component (the camera). This will ensure that the text will always face the user. Ensure that you have linked the source code of the A-Frame `look-at` component! 
    - Set the text's value to the peak's name. In other words, the name of each peak will be displayed.
    - Scale the text appropriately. 
    - Add the `gps-projected-entity-place` component to our `a-text` to position the text using latitude and longitude. The coordinates are extracted from the GeoJSON.
    - Using standard DOM, append the `a-text` to the parent element of our `peakfinder` component.

Try this out, making sure you do so in an area where there are at least small hills, which are likely to be on OpenStreetMap. You can check [OpenStreetMap](https://openstreetmap.org) and look for peak symbols near you on the map. If necessary, expand the bounding box.

You will find that the text should appear at the correct place for the peak, **but** with one big problem - elevation is not included. So far, we are only using latitude and longitude to place the peak. Clearly, a fully-working peakfinder app will include elevation too! The next tutorial section addresses this. 

Nonetheless, even without elevation, hopefully this tutorial will show you how easy it is to generate dynamic location-based AR content using AR.js.

### Things to try

- Rather than hard-coding the scale at 2000, add it as a property using the component's schema, and experiment with different values by setting the property in your HTML.


## Adding elevation, and tiling

As discussed, we would like our peakfinder app to show the peaks at the correct *elevation*. Another issue with any outdoor AR app is that we want, ideally, to *download new data as we move to new areas* so that, for example, peaks within 10km of us are downloaded when we start the app, and then as we reach the edge of the already-downloaded area, new peaks are downloaded. At the moment, we simply download the peaks when we start the app, and do not update them.

These issues provide much of the complexity of a location-based AR app. Luckily, though, a pre-built solution to both problems exists. One of the great things about A-Frame is the fact that there are *many pre-built components which can be added to our app*, and two pre-built components exists for this precise problem: `terrarium-dem` and `osm3d`, both part of the [aframe-osm-3d](https://github.com/nickw1/aframe-osm-3d) package.

### How the aframe-osm-3d components work

- `terrarium-dem` downloads Digital Elevation Model (DEM) data for a given location. This data was based on NASA SRTM data, and was converted into the Terrarium PNG format by the former mapping company Mapzen (see [original article](https://www.mapzen.com/blog/terrain-tile-service/)). As discussed in this article, elevation is encoded in the red, green, and blue channels of a PNG image, and is now available, without usage restrictions, on [AWS](https://registry.opendata.aws/terrain-tiles/). The raw elevation data is emitted within an event (see below) and optionally, the terrain is rendered as a 3D mesh.

- `osm3d` is used to download OSM data from a GeoJSON data structure in 3D, in other words, the coordinates of the OSM features contain not just latitude and longitude, but elevation in metres. The elevation is obtained from the DEM emitted from `terrarium-dem`, above; `osm3d` automatically listens for the `terrarium-dem-loaded` event emitted from `terrarium-dem` and uses the DEM to calculate the elevation.

You can find out how these components work in more detail by visiting their [GitHub repository](https://github.com/nickw1/aframe-osm-3d).


### Tiling

The `aframe-osm-3d` components discussed above both work using a **tiling** system. Regions of the world are split up into *tiles*, using the "XYZ" or "Google" tiling system, which you can read about in more detail [here](https://developers.google.com/maps/documentation/javascript/coordinates). The general idea is that each tile is defined by an X, a Y, and a Z coordinate, in which:

- X represents the tile's coordinate on a west-east axis; 
- Y represents the tile's coordinate on a north-south axis; 
- Z represents the tile's zoom level, discussed in more detail in the link above.

`terrarium-dem` reads in a given longitude and latitude, and downloads the current XYZ elevation tile at a given zoom level, *and* the eight surrounding tiles (nine in total). These tiles are internally cached by `terrarium-dem` so that if a nearby location is requested, and the user hasn't moved to a different tile, the cached data will be used, avoiding unnecessary downloads. Also, when tiles are downloaded, they are emitted within the `terrarium-dem-loaded` event, which the `osm3d` component responds to by downloading the corresponding tiles from an OSM web API, calculating the OSM data's elevation using the DEM, and again caching and emitting the result.

### The code

With that in mind, here is our code which makes use of `terrarium-dem` and `osm3d` to download elevation and OSM data to create peaks with elevations. We will start with the HTML:

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8" />
<title>AR.js Peak Finder</title>
<script src="https://aframe.io/releases/1.0.4/aframe.min.js"></script>
<script src="https://unpkg.com/aframe-look-at-component@0.8.0/dist/aframe-look-at-component.min.js"></script>
<script src="https://raw.githack.com/AR-js-org/AR.js/master/aframe/build/aframe-ar-nft.js"></script>
<script src="js/bundle.js"></script>
</head>
<body>
    <a-scene
        vr-mode-ui="enabled: false"
        arjs='sourceType: webcam; videoTexture: true; debugUIEnabled: false;'>
    <a-camera gps-projected-camera rotation-reader></a-camera>
    <a-entity terrarium-dem='zoom: 12; url: proxy.php?x={x}&y={y}&z={z}' osm3d='url: https://hikar.org/fm/ws/tsvr.php?x={x}&y={y}&z={z}&amp;poi=natural&amp;outProj=4326' peakfinder></a-entity>
</a-scene>
</body>
</html>
```

What's new here?

- Note that we are linking in our JavaScript (which we will come to shortly) as a *bundle*. This is a great way of developing code which relies on third-party modules without having to link multiple JS files, and we will expand upon this below.
- Note how our custom entity now has some additional components: the `terrarium-dem` and `osm3d` components which we discussed above. Note how each component has a `url` property defining the source URL to download the data.
- Note also how we set the `zoom` to 12 for `terrarium-dem`. This will use the zoom level of 12 for our tiles, which means that peaks several kilometres away in every direction will be downloaded on startup.

You'll notice that the `url` property for `terrarium-dem` is not an AWS URL but rather a local URL, `proxy.php`. Why is this? Due to the [same-origin-policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy) we cannot contact AWS directly via AJAX, so we need to create a proxy script to do it for us. The example provided is in PHP and will look something like the code below. (If your server does not have PHP installed, feel free to replace this code with another language).

```php
<?php
$x = $_GET["x"];
$y = $_GET["y"];
$z = $_GET["z"];
if(ctype_digit($x) && ctype_digit($y) && ctype_digit($z)) {
    header("Content-Type: image/png");
    echo file_get_contents("https://s3.amazonaws.com/elevation-tiles-prod/terrarium/$z/$x/$y.png");
} else {
    header("HTTP/1.1 400 Bad Request");
    header("Content-Type: application/json");
    echo json_encode(["error" => "invalid x, y and/or z params"]);
}
?>
```

For the OSM URL (on `hikar.org`), we do not need a proxy as this script has [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) enabled by default.

We can now move on to our JavaScript. This will look something like this:
```javascript
require('aframe-osm-3d');

AFRAME.registerComponent('peakfinder', {
    schema: {
        scale: {
            type: 'number',
            default: 15
        }
    },

    init: function() {
        this.textScale = this.data.scale * 100;
        this.camera = document.querySelector('a-camera');

        window.addEventListener('gps-camera-update-position', e => {
            this.el.setAttribute('terrarium-dem', {
                lat: e.detail.position.latitude,
                lon: e.detail.position.longitude 
            })
        });

        this.el.addEventListener('elevation-available', e => {
            const position = this.camera.getAttribute('position');
            position.y = e.detail.elevation + 1.6;
            this.camera.setAttribute('position', position);
        });

        this.el.addEventListener('osm-data-loaded', e => {
            e.detail.pois
                .filter ( f => f.properties.natural == 'peak' )
                .forEach ( peak => {
                    const entity = document.createElement('a-entity');
                    entity.setAttribute('look-at', '[gps-projected-camera]');
                    const text = document.createElement('a-text');
                    text.setAttribute('value', peak.properties.name);
                    text.setAttribute('scale', {
                        x: this.textScale,
                        y: this.textScale,
                        z: this.textScale
                    });
                    text.setAttribute('align', 'center');
                    text.setAttribute('position', {
                        x: 0,
                        y: this.data.scale * 20, 
                        z: 0
                    });
                    entity.setAttribute('gps-projected-entity-place', {
                        latitude: peak.geometry.coordinates[1],
                        longitude: peak.geometry.coordinates[0]
                    });
                    entity.setAttribute('position', {
                        x: 0,
                        y: peak.geometry.coordinates[2],
                        z: 0
                    });
                    entity.appendChild(text);
                    const cone = document.createElement('a-cone');
                    cone.setAttribute('radiusTop', 0.1);
                    cone.setAttribute('scale', {
                        x: this.data.scale * 10,
                        y: this.data.scale * 10,
                        z: this.data.scale * 10
                    });
                    cone.setAttribute('height', 3);
                    cone.setAttribute('material', { color: 'magenta' } );
                    entity.appendChild(cone);

                    this.el.appendChild(entity);
                });
        });
    }
});
```

How does this work?

- We start by *require*ing the `aframe-osm-3d` package. This is a package available on [NPM](https://npmjs.com), and as we have seen, contains the `terrarium-dem` and `osm3d` components. `aframe-osm-3d` is a JavaScript *module*, specifically a CommonJS module, and this technique of *require*ing modules is standard in Node.js. However with the help of tools such as [Browserify](https://browserify.org), we can use modules in the browser, too. More on this below.

- We then begin our actual component. Note how its schema includes a `scale`, allowing us to control the scale of the rendered peaks. Feel free to experiment with changing this.

- We then move on to our `init()`. This works a bit differently this time:
	- We once again handle the `gps-camera-update-position` event, but this time we pass the latitude and longitude onto our `terrarium-dem` component. As we saw above, this will download the local tiles for this position.
	- We next handle the `elevation-available` event, which is emitted from our `terrarium-dem` component as soon as we have elevation available for our current position. The elevation is contained in the emitted event; we handle it by setting the camera's `y` coordinate to that elevation plus 1.6 (to account for the fact that a user will be holding the device above the ground0.
	- We then handle the `osm-data-loaded` event. This will be emitted from `osm3d` whenever new data has been downloaded from the OSM API and elevation added. This event contains various properties, but we will use the `pois` property which is an array of GeoJSON features, one for each POI. Note how we filter the POIs to select peaks only, once again.
	- We then create one entity per peak, as before. We ensure the entity is looking at the camera, and then create two child entities, an `a-cone` primitive to show a peak-like graphic, and an `a-text` primitive to show the peak's name. We align the text to the centre so that it's correctly placed relative to the cone, and set coordinates and scale appropriately. We also add a `gps-projected-entity-place` component to each entity, as before, so that we can position them using their latitude and longitude.
	- Note how, once we have set the latitude and longitude with `gps-projected-entity-place`, we **set the elevation** of each peak entity. We do this using the third member of the `coordinates` array contained within the GeoJSON, which will contain the peak's elevation in metres.

And that is all that is needed to create a very simple peak finder in AR.js!

We now need to *build* the peakfinder, as indicated above. First, we need to
install the `aframe-osm-3d` package with NPM (included with [Node.js](https://nodejs.org)).
```
npm install aframe-osm-3d
```
Next, we need to create a *bundle* with [Browserify](https://browserify.org). Browserify is an example of a "bundling" tool. It takes multiple JavaScript files, including third-party modules, and produces a single **bundle**, which can be linked to from an HTML page. To use, it's simply:
```
browserify peakfinder.js > bundle.js
```
(assuming that you saved your peakfinder component as `peakfinder.js`).

And that is it! Try it out and see if you can see peaks in their correct location, and elevation.

### Further things to try

It would be nice to show users status messages as the peaks are downloading. You can do this by handling a couple of other events:

- `terrarium-start-update` is emitted when we start to download new DEM data.
- `terrarium-dem-loaded`, as we have already seen, is emitted when DEM data has finished downloading.

Adapt your example to include a `<div>` which displays a status message. (You can make a `<div>` a child of your `<a-scene>`). Display "Downloading elevation data..." while it's downloading the DEM data, and then "Downloading OSM data" when the DEM download has finished. Finally, when the OSM data has loaded, set the `<div>`'s contents to a blank string. 
