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
| simulateLatitude   | Setting this allows you to simulate the latitude of the camera, to aid in testing.    | 0 (disabled) |
| simulateLongitude   | Setting this allows you to simulate the longitude of the camera, to aid in testing.    | 0 (disabled) |
| simulateAltitude   | Setting this allows you to simulate the altitude of the camera in meters above sea level, to aid in testing.    | 0 (disabled) |


### `gps-entity-place`

**Required**: yes

**Max allowed per scene**: no limit

This component makes each entity GPS-trackable. This assigns a specific world position to an entity, so that the user can see it when their device is pointing to its position in the real world. If the user is far from the entity, it will seem smaller. If it's too far away, it won't be seen at all.

It requires latitude and longitude as a single string parameter (example with `a-box` aframe primitive):

```HTML
<a-box color="yellow" gps-entity-place="latitude: <your-latitude>; longitude: <your-longitude>"/>
```

⚡️ In addition, you can use the a-frame "position" parameter to assign a y-value to change the height of the content.
This value should be entered as meters above or below (if negative) the current camera height. For example, this would assign a height of 30 meters, and will be displayed relative to the gps-camera's current height:

```HTML
<a-box color="yellow" gps-entity-place="latitude: <your-latitude>; longitude: <your-longitude>" position="0 30 0"/>
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


