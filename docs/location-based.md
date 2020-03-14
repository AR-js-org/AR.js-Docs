# Location Based

Location Based has been implemented only for A-Frame framework.
Here's the API Reference.

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

In addition, you can use the a-frame "position" parameter to assign a y-value to change the height of the content.
This value should be entered in meters above or below sea level. For example, this would assign a height of 300 meters above sea level, and will be displayed relative to the gps-camera's current altitude:

```HTML
<a-box color="yellow" gps-entity-place="latitude: <your-latitude>; longitude: <your-longitude>" position="0 300 0"/>
```

### Properties

No real property apart from the string that defined latitude and longitude together, as shown above.

### Custom Attributes

The following are Custom Attributes that can be retrieved from `gps-entity-place` entities, for example:

```js
const distanceMsg = document.querySelectr('[gps-entity-place]').getAttribrute('distanceMsg');
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


⚡️ Usually, in Location Based, it's nice to have the augmented content that will always face the user, so when you rotate the camera, 3D models or most of all, text, is well visible.

Look at [this example](https://github.com/AR-js-org/AR.js/tree/master/aframe/examples/location-based/always-face-user) in order to create `gps-entity-place` entities that will always face the user (camera).


