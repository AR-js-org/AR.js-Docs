# Location Based

Location Based has been implemented only for A-Frame framework.

[This article ](https://medium.com/chialab-open-source/build-your-location-based-augmented-reality-web-app-c2442e716564) gives you a first glance to Location Based on AR.js.
It can be used for indoor (but with low precision) and outdoor geopositioning of AR content.

You can load places statically, from HTML or from Javascript, or you can load your data from local/remote json, or even through API calls. Choice is yours. On the article above there are all the options explained, as tutorials.

Following there's the API Reference.

### `gps-camera`

**Required**: yes

**Max allowed per scene**: 1

This component enables the Location AR. It has to be added to the `camera` entity.
It makes possible to handle both position and rotation of the camera and it's used to determine where the user is pointing their device.

For example:

```HTML
<a-camera gps-camera rotation-reader></a-camera>
```

In addition to that, as you can see on the example above, we also have to add `rotation-reader` to handle rotation events. See [here](https://aframe.io/docs/0.9.0/components/camera.html#reading-position-or-rotation-of-the-camera) for more details.


### Properties

| Property   | Description | Default Value |
|------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|
| alert     | Whether to show a message when GPS signal is under the `positionMinAccuracy`                  | false |                                                                                                                                                                        | true          |
| positionMinAccuracy        | Minimum accuracy allowed for position signal    | 100 |
| minDistance        | If set, places with a distance from the user lower than this value, are not showed. Only a positive value is allowed. Value is in meters.    | 0 (disabled) |
| maxDistance        | If set, places with a distance from the user higher than this value, are not showed. Only a positive value is allowed. Value is in meters.    | 0 (disabled) |
| simulateLatitude   | Setting this allows you to simulate the latitude of the camera, to aid in testing.    | 0 (disabled) |
| simulateLongitude   | Setting this allows you to simulate the longitude of the camera, to aid in testing.    | 0 (disabled) |
| simulateAltitude   | Setting this allows you to simulate the altitude of the camera in meters above sea level, to aid in testing.    | 0 (disabled) |
| gpsMinDistance   | Setting this allows you to control how far the camera must move, in meters, to generate a GPS update event. Useful to prevent 'jumping' of augmented content due to frequent small changes in position.    | 5 |
| gpsTimeInterval   | Setting this allows you to control how frequently to obtain a new GPS position. If a previous GPS location is cached, the cached position will be used rather than a new position if its 'age' is less than this value, in milliseconds. This parameter is passed directly to the Geolocation API's `watchPosition()` method.    | 0 (always use new position, not cached) |


### `gps-entity-place`

**Required**: yes

**Max allowed per scene**: no limit

This component makes each entity GPS-trackable. This assigns a specific world position to an entity, so that the user can see it when their device is pointing to its position in the real world. If the user is far from the entity, it will seem smaller. If it's too far away, it won't be seen at all.

It requires latitude and longitude as a single string parameter (example with `a-box` aframe primitive):

```HTML
<a-box material="color: yellow" gps-entity-place="latitude: <your-latitude>; longitude: <your-longitude>"/>
```

⚡️ In addition, you can use the a-frame "position" parameter to assign a y-value to change the height of the content.
This value should be entered as meters above or below (if negative) the current camera height. For example, this would assign a height of 30 meters, and will be displayed relative to the gps-camera's current height:

```HTML
<a-box material="color: yellow" gps-entity-place="latitude: <your-latitude>; longitude: <your-longitude>" position="0 30 0"/>
```

### Properties

No real property apart from the string that defined latitude and longitude together, as shown above.

### Custom Attributes

The following are Custom Attributes that can be retrieved from `gps-entity-place` entities, for example:

```js
const distanceMsg = document.querySelector('[gps-entity-place]').getAttribute('distanceMsg');
console.log(distanceMsg);   // "890 meters"
```

| Custom Attribute   | Description | Default Value |
|------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|
| distance        | Distance from user, updated at every user position update. Value in meters.   | 0 |
| distanceMsg        | Distance from user as string, with unit, updated at every user position update. Value as `<distance> meters/kilometers`.   | '' |

----

### Events

Take a look at the [UI and Events](./ui-events.md) page for Location Based Custom Events.

-----

⚡️ Usually, in Location Based, it's nice to have the augmented content that will always face the user, so when you rotate the camera, 3D models or most of all, text, are well visible.

Look at [this example](https://github.com/AR-js-org/AR.js/tree/master/aframe/examples/location-based/always-face-user) in order to create `gps-entity-place` entities that will always face the user (camera).


## Projected Camera Version

The experimental 'projected camera' version of the location-based components for AR.js uses [Spherical Mercator](https://www.maptiler.com/google-maps-coordinates-tile-bounds-projection) (aka EPSG:3857) to store both the camera position and the position of added points of interest and other geographical data.

The rationale for this version is to allow easy addition of more complex geographic data such as roads and paths. Such data can be projected and added to an AR.js scene, and then, because Spherical Mercator units approximate to metres (away from the poles), the coordinates can be used directly as WebGL/A-Frame world coordinates.

The two components for the projected camera version are `gps-projected-camera` and `gps-projected-entity-place`. Their interface is almost identical to `gps-camera` and `gps-entity-place` but they work differently internally.

For example:

```HTML
<a-camera gps-projected-camera rotation-reader></a-camera>
```
and:

```HTML
<a-box color="yellow" gps-projected-entity-place="latitude: <your-latitude>; longitude: <your-longitude>"/>
```
Note that internally, the latitude and longitude are converted to Spherical Mercator coordinates.

As for `gps-entity-place`, you can specify an altitude using the *y* component of the `position` attribute:

```HTML
<a-box color="yellow" gps-projected-entity-place="latitude: <your-latitude>; longitude: <your-longitude>" position="0 30 0"/>
```

### Calculating world coordinates of arbitrary augmented content

`gps-projected-camera` has some useful properties and methods which can be used to easily work with arbitrary augmented content (for example, polylines or polygons sourced from geodata APIs such as [OpenStreetMap](https://openstreetmap.org)). Before introducing these, it needs to be made clear that, in `gps-projected-camera`, the **original GPS position** is set as the world origin. So, if arbitrary content is to be added to the scene, and the source coordinates for this content is in unprojected (WGS84) latitude and longitude, it needs to be:

- projected to Spherical Mercator;
- and then converted to world coordinates relative to the original GPS position.

On the other hand, only the second step is needed if the source coordinates are already projected. We'll look at each scenario now.

#### Source data in WGS84 latitude/longitude

This is probably the most common scenario.
The `latLonToWorld(lat, lon)` method of the `gps-projected-camera` component converts latitude and longitude directly to world coordinates, performing the projection as the first step and then calculating the world coordinates from the projected coordinates. It will return a 2-member array containing the *x* and *z* world coordinates, allowing the developer to calculate or specify the *y* coordinate (altitude) independently.

#### Source data in Spherical Mercator

An alternative scenario is when the augmented content has already been projected into Spherical Mercator and therefore does not need the initial projection step when added to an AR.js scene. This may occur when an API serves data in Spherical Mercator, for instance. In this case, we still need to convert the Spherical Mercator coordinates to world coordinates relative to the original GPS position. `gps-projected-camera` has an `originCoordsProjected` property, which represents the original GPS position in Spherical Mercator coordinates. This is a two-member array, containing the Spherical Mercator easting and northing, respectively, of the origin point. From this, we can therefore work out the world coordinates from Spherical Mercator coordinates:

* `xWorld = featureEasting - originCoordsProjected[0]`

and

* `zWorld = -(featureNorthing - originCoordsProjected[1])`

where `xWorld` and `zWorld` are the world *x* and *z* coordinates of the augmented content, and `featureEasting` and `featureNorthing` are the content's Spherical Mercator coordinates. Note how we have to reverse the sign of *z* as increasing Spherical Mercator easting corresponds to increasing *x* in OpenGL coordinates, and increasing altitude corresponds to increasing *y*, but increasing Spherical Mercator northing corresponds to **decreasing** *z*.
