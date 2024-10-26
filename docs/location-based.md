# Location Based

**Important!** You might want to check out the new AR.js [LocAR](https://github.com/AR-js-org/locar.js) project if you are interested in location-based AR. This aims to provide a cleaner API, with just a single version, and more frequent updates. 

In the future, updates on the location-based side will be focused on LocAR.

## Intro to location-based 

[This article](https://medium.com/chialab-open-source/build-your-location-based-augmented-reality-web-app-c2442e716564) gives you a first glance to Location Based on AR.js.

It can be used for indoor (but with low precision) and outdoor geopositioning of AR content.

You can load places statically, from HTML or from Javascript, or you can load your data from local/remote json, or even through API calls. Choice is yours. On the article above there are all the options explained, as tutorials.

Location Based has been implemented for both three.js and A-Frame. Each of these is documented below.

This document is intended as reference documentation. There are also two *tutorials* available, with full example code:

- [A-Frame location based](location-based-aframe/index.md)
- [three.js location based](location-based-three/index.md)

## Limitations

Location-based AR with AR.js is subject to certain limitations.

- Your device must have a GPS chip, accelerometer and magnetometer.
- On some devices, the sensors may be miscalibrated, resulting in an incorrect North. See, for example, [this three.js issue](https://github.com/mrdoob/three.js/pull/22654). This is unfortunately a limitation of the device. This will be investigated further in [LocAR](https://github.com/AR-js-org/locar.js), for example, as to whether certain devices are consistently "out" by a certain bearing.
- The camera feed may appear "stretched". Again the focus on fixing this will be in LocAR.

## A-Frame

AR.js offers A-Frame components to implement location-based AR. There are three variants of the components, detailed as below:

- The `new-location-based` components. **In most cases, these are recommended**. These have been available since AR.js 3.4.0, incorporate various bug fixes, use simpler code, and provide a thin wrapper round the three.js API shown below. These are recommended for most uses, though do not support all the [events](./ui-events.md) of the older components due to a different internal implementation. Nonetheless they the components likely to see further development - the older variants are unlikely to see further work besides bug fixes.

- The `projected` components. These have been available since AR.js 3.3.1, use largely the same internal implementation as the classic components, and were the first to offer projection of latitude/longitude into Spherical Mercator, discussed below. They are generally **not recommended** unless you have problems with `new-location-based`.

- The classic components, available before AR.js 3.3.1. These are similar to the `projected` components but do not offer the facility to convert between latitude/longitude and the projected coordinates used for augmented reality, which can cause problems for more specialist uses such as showing roads and paths in augmented reality. For most use cases it is preferred to use `new-location-based` but some uses, such as embedded AR scenes, only work with the classic components.

### The components

Each variant above includes two components, a `camera` component which enables the location-based AR, and an `entity-place` component which enables setting components' latitude and longitude. The exact component names for each variant are shown below.

|Component variant|Camera component|Entity-place component
|-----------------|----------------|----------------------
|new-location-based|gps-new-camera|gps-new-entity-place
|projected|gps-projected-camera|gps-projected-entity-place
|classic|gps-camera|gps-entity-place

### Camera component (`gps-new-camera`, `gps-projected-camera` or `gps-camera`)

**Required**: yes

**Max allowed per scene**: 1

This component enables the Location AR. It has to be added to the `camera` entity.
It makes possible to handle both position and rotation of the camera and it's used to determine where the user is pointing their device.

For example:

```HTML
<a-camera gps-new-camera></a-camera>
```

### Properties

| Property   | Description | Default Value | Availability
|------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|--------
| positionMinAccuracy        | Minimum accuracy allowed for position signal    | 100 | all
| gpsMinDistance | Setting this allows you to control how far the camera must move, in meters, to generate a GPS update event. Useful to prevent 'jumping' of augmented content due to frequent small changes in position.    | 5 | all 
| simulateLatitude   | Setting this allows you to simulate the latitude of the camera, to aid in testing.    | 0 (disabled) | all (but only triggers GPS update event in new-location-based)
| simulateLongitude   | Setting this allows you to simulate the longitude of the camera, to aid in testing.    | 0 (disabled) | all (but only triggers GPS update event in new-location-based)
| simulateAltitude   | Setting this allows you to simulate the altitude of the camera in meters above sea level, to aid in testing.    | 0 (disabled) | all
| alert     | Whether to show a message when GPS signal is under the `positionMinAccuracy`                  | false | projected, classic 
| minDistance        | If set, places with a distance from the user lower than this value, are not shown. Only a positive value is allowed. Value is in meters. **In the `new-location-based` components, please set the `near` clipping plane of the perspective camera.**    | 0 (disabled) | projected, classic
| maxDistance        | If set, places with a distance from the user higher than this value, are not shown. Only a positive value is allowed. Value is in meters. **In the `new-location-based` components, please set the `far` clipping plane of the perspective camera.**   | 0 (disabled) | projected, classic
| gpsTimeInterval   | Setting this allows you to control how frequently to obtain a new GPS position. If a previous GPS location is cached, the cached position will be used rather than a new position if its 'age' is less than this value, in milliseconds. This parameter is passed directly to the Geolocation API's `watchPosition()` method.    | 0 (always use new position, not cached) | all


### Entity-place component (`gps-new-entity-place`, `gps-projected-entity-place` or `gps-entity-place`)

**Required**: yes

**Max allowed per scene**: no limit

This component makes each entity GPS-trackable. This assigns a specific world position to an entity, so that the user can see it when their device is pointing to its position in the real world. If the user is far from the entity, it will seem smaller. If it's too far away, it won't be seen at all.

It requires latitude and longitude as a single string parameter (example with `a-box` aframe primitive):

```HTML
<a-box material="color: yellow" gps-new-entity-place="latitude: <your-latitude>; longitude: <your-longitude>"/>
```

⚡️ In addition, you can use the a-frame "position" parameter to assign a y-value to change the height of the content.
This value should be entered as meters above or below (if negative) the current camera height. For example, this would assign a height of 30 meters, and will be displayed relative to the gps-new-camera's current height:

```HTML
<a-box material="color: yellow" gps-new-entity-place="latitude: <your-latitude>; longitude: <your-longitude>" position="0 30 0"/>
```

### Properties

- `distance` : current distance from the camera, in metres. Available in `gps-new-entity-place` only: for the classic components, please use [events](./ui-events.md) to obtain the current distance.

----

### Events

Take a look at the [UI and Events](./ui-events.md) page for Location Based Custom Events.

----

⚡️ Usually, in Location Based, it's nice to have the augmented content that will always face the user, so when you rotate the camera, 3D models or most of all, text, are well visible.

Look at [this example](https://github.com/AR-js-org/AR.js/tree/master/aframe/examples/location-based/always-face-user) in order to create `gps-new-entity-place` entities that will always face the user (camera).

----

## Viewing every distant object

If your location-based AR content is distant from the user (around 1km or
more), it is recommended to use the new `arjs-webcam-texture` component (introduced in AR.js 3.2.0), which uses a three.js texture to stream the camera feed and allows distant content to be viewed. This component is automatically injected if the `videoTexture` parameter of the `arjs` system is set to `true` *and* the `sourceType` is `webcam`. For example (code snippet only):

```html
    <a-scene
      vr-mode-ui="enabled: false"
      arjs="sourceType: webcam; videoTexture: true; debugUIEnabled: false;"
    >
```

## Reducing shaking effects

In location-based mode, 
'shaking' effects can occur due to frequent small
changes in the device's orientation, due to the high sensitivity of the device
sensors such as the accelerometer. 

If using AR.js 3.3.1 or greater (3.4.3 or greater for the `new-location-based` components), this can optionally be reduced using an exponential smoothing technique. *Note that, if you are NOT using the `new-location-based` components, there are currently some occasional display artefacts with this if moving the device quickly or suddenly so please test before you enable it in a finished application; work to resolve these is on-going. Alternatively, please use the `new-location-based` components.*

This is enabled by adding a custom `look-controls` component to your `a-camera` with a `smoothingFactor` property. This replaces A-Frames default `look-controls` component, which must be disabled. 

The name of the custom `look-controls` component varies, depending on which version of the `location-based` components you are using:

- for `new-location-based`, use `arjs-device-orientation-controls`;
- for the classic and projected components, use `arjs-look-controls`.

For example, in the `new-location-based` components:

```html
<a-camera id='camera1' look-controls-enabled='false' arjs-device-orientation-controls='smoothingFactor: 0.1' gps-new-camera='gpsMinDistance: 5'> </a-camera>
```

or, otherwise:

```html
<a-camera id='camera1' look-controls-enabled='false' arjs-look-controls='smoothingFactor: 0.1' gps-projected-camera='gpsMinDistance: 5'> </a-camera>
```

Exponential smoothing works by applying a smoothing factor to each newly-read device rotation angle (obtained from sensor readings) such that the previous smoothed value counts more than the current value, thus reducing 'noise' and 'jitter'. If `k` is the smoothing factor:

```
smoothedAngle = k * newValue + (1 - k) * previousSmoothedAngle
```

It can be seen from this that the **smaller** the value of `k` (the `smoothingFactor` property), the **greater** the smoothing effect. In tests, 0.1 appears to give the best result.

You can also reduce 'jumping' of augmented content when near a place - a bad-looking effect due to GPS sensor's low precision. To do so you can use the `gpsMinDistance` property, as shown in the examples above. This will only update the position if the user has moved at least that number of metres.

----

## Projection Details

The `new-location-based` and `projected` location-based components for AR.js uses [Spherical Mercator](https://www.maptiler.com/google-maps-coordinates-tile-bounds-projection) (aka EPSG:3857) to store both the camera position and the position of added points of interest and other geographical data.

Spherical Mercator is the same projection used by Google Maps and projects the earth onto a flat surface. It works reasonably at most latitudes but is highly distorted near the poles. Latitude and longitude is projected into Spherical Mercator *eastings* and *northings*, which are approximately (but not exactly) equivalent to metres. 

The rationale for this is to allow easy addition of more complex geographic data such as roads and paths. Such data can be projected and added to an AR.js scene, and then, because Spherical Mercator units approximate to metres (away from the poles), the coordinates can be used directly as WebGL/A-Frame world coordinates.

### Calculating world coordinates of arbitrary augmented content

The `new-location-based` and `projected` components have some useful properties and methods which can be used to easily work with more specialist augmented content (for example, you might want to overlay AR polylines or polygons representing roads and paths, downloaded from geodata APIs such as [OpenStreetMap](https://openstreetmap.org)).  Such data can be downloaded from the API as lat/lon based coordinates, projected using AR.js API methods into Spherical Mercator (approximating to, but not exactly metres, but in tests good enough to use as world coordinates), and then added to the scene as a three.js object.

This is implemented differently in the `new-location-based` and `projected` components, but the external API is (as of 3.4.3) the same.

The key method is the `latLonToWorld(lat, lon)` method of the `gps-new-camera` and `gps-projected-camera` components. This converts latitude and longitude directly to world coordinates, performing the projection as the first step and then calculating the world coordinates from the projected coordinates. It will return a 2-member array containing the *x* and *z* world coordinates, allowing the developer to calculate or specify the *y* coordinate (altitude) independently.

Note that the sign of the Spherical Mercator northing is reversed to align with the OpenGL coordinate system (eastings are equivalent to `x` coordinates and northings to `z` coordinates).

`gps-new-camera` implements projection via the underlying AR.js three.js `LocationBased` object (see three.js documentation, below) which is responsible for the actual projection. 

`gps-projected-camera` provides similar functionality but via a different method and with some implementation differences. In `gps-projected-camera`, unlike `gps-new-camera`, the **original GPS position** is set as the world origin.


## three.js 

For pure three.js (no A-Frame) it is recommended to use [LocAR](https://github.com/AR-js-org/locar.js). The notes below, however, refer to the three.js version in the main AR.js repository.

The three.js API keeps track of your current GPS location (or allows you to set a fake location) and allows you to add three.js objects at a given latitude and longitude. It includes these classes:

- `THREEx.LocationBased` - general manager class for the three.js location-based API.
- `THREEx.WebcamRenderer` - renders the feed from the webcam as a WebGL texture.
- `THREEx.DeviceOrientationControls` - for detecting changes in the orientation of the device.

These classes include the following methods:

### LocationBased

- `constructor(scene, camera, options={})` : Initialises a new `LocationBased` object. Takes a `THREE.Scene` and a `THREE.Camera` object as parameters, as well as an object of GPS options (see `setGpsOptions()`, below)

- `setProjection(proj)` : allows the projection to be defined. By default Spherical Mercator is used. The projection object must provide a `project()` method which takes longitude and latitude as parameters and returns a 2-member array of projected coordinates (easting, northing).

- `setGpsOptions(options={})` : sets the GPS options. These include `gpsMinDistance` and `gpsMinAccuracy`, described in the A-Frame documentation above.

- `startGps()` : starts the GPS. Takes an optional `maximumAge`, as used by the native Geolocation API.

- `stopGps()` : stops the GPS.

- `fakeGps(lon, lat, elev=null, acc=0)` : fakes a GPS position being received. Elevation and accuracy can optionally be provided.

- `lonLatToWorldCoords(lon, lat)` : projects a given longitude and latitude into world coordinates using the current projection. The sign of the northing is reversed to align it with the OpenGL coordinate system.

- `add(object, lon, lat, elev)` : adds a given three.js object to the world at the given longitude and latitude and at the given elevation.

- `setWorldPosition(object, lon, lat, elev)` : changes the world position of a given object to the given longitude and latitude, without adding it to the scene.

- `setElevation(elev)` : sets the current elevation in metres. This will set the camera's `y` coordinate to that elevation.

- `on(eventname, eventhandler)` : allows event handlers to be specified. Currently `gpsupdate` and `gpserror` handlers are supported, for receiving a new GPS position and GPS errors (as in the Geolocation API) respectively.

### WebcamRenderer

Renders the webcam feed.

- `constructor(renderer, videoElementSelector)` : creates a `WebcamRenderer`. Takes a `THREE.WebGLRenderer` plus a selector for an HTML `video` element to stream the feed to.

- `update()` : updates the camera feed. Should be done each time the scene is rendered.

### DeviceOrientationControls

Represents the device orientation controls, i.e. accelerometer and magnetic field sensors, for determining the orientation of the device. Based on the sample included in the three.js distribution.

- `constructor(cameraObject)` : creates a `DeviceOrientationControls` object. Takes a three.js camera.

- `update()` : updates the device orientation controls. Should be done each time the scene is rendered.

### Using three.js location-based in an application

You are recommended to use `npm` to install AR.js, `import` it into your application, and use a bundler such as Webpack to build.

Here is a sample `package.json`:

```
{
    "dependencies": {
        "@ar-js-org/ar.js": "3.4.5",
    },
    "devDependencies": {
        "webpack": "^5.75.0",
        "webpack-cli": "^5.0.0"
    },
    "scripts": {
        "build": "npx webpack"
    }
}
```
and a sample `webpack.config.js`:
```javascript
const path = require('path');

module.exports = {
    mode: 'development',
    entry: './index.js',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'bundle.js'
    },
    optimization: {
        minimize: false
    }
};
```
This will build a bundle named `bundle.js` in the `dist` subdirectory from a source file `index.js`. 

Here is an example of importing the components into an application:
```javascript
import * as THREEx from './node_modules/@ar-js-org/ar.js/three.js/build/ar-threex-location-only.js'
```
