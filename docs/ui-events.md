# UI and Custom Events

To make AR.js based Web App looking better and add UI capabilities, it's possible to treat is as common website.
Here you will learn how to use Raycaster, Custom Events and Interaction with overlayed DOM elements.

First of all, an introduction about click interactions: it is possible to click directly on content using A-Frame, we tested and had good result with version `0.9.2`. Keep in mind that clicks (or touch interactions) are not very reliable: it can work if you

- need only few clicks, not very reliable
- have to click on very few entities on screen
- clicks work only on the center of the screen, not at the angles, so beware that your entities, to be clicked, should be big enough and not positioned on angles.

Always remember that AR.js is Web AR, that means, you can use **any** DOM elements interactions you want.
Such interactions work perfectly and most of the times, the Overlayed DOM content interaction is enough for common needs.

## Handle clicks on AR content

We can use click/touch interactions through Raycaster, following [A-Frame Docs](https://aframe.io/docs/1.0.0/introduction/interactions-and-controllers.html).

We can register an event listener for any a-frame entity. So this method works also for Marker Based, Image Tracking and Location Based.
We will show it on a Location Based examples, registering the click on `a-entity` - the same can be done with `a-marker` or `a-nft`.

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <title>Click Location Based Example</title>
    <script src="https://aframe.io/releases/0.9.2/aframe.min.js"></script>
    <script src="https://raw.githack.com/AR-js-org/AR.js/master/aframe/build/aframe-ar-nft.js"></script>
  </head>

  <script>
    // here we register che 'clickhandler' component
    AFRAME.registerComponent("clickhandler", {
      init: function() {
        // we add the click event listener to any instance of this component
        this.el.addEventListener("click", () => {
          alert("Clicked!");
        });
      }
    });
  </script>

  <body style="margin: 0; overflow: hidden;">
    <!-- with cursor and raycaster, we define a raycaster handler.
    With 'raycaster' we add a CSS selector to match the only elements that will react to click events,
    for performances reasons. See: (https://aframe.io/docs/1.0.0/introduction/interactions-and-controllers.html) -->
    <a-scene
      cursor="rayOrigin: mouse; fuse: true; fuseTimeout: 0;"
      raycaster="objects: [clickhandler];"
      vr-mode-ui="enabled: false"
      embedded
      arjs="sourceType: webcam; debugUIEnabled: false;"
    >
      <!-- this entity, potentially every other on the a-scene, has the 'clickhandler' component
      and will behave has we have defined: it will handle click events! -->
      <a-box
        clickhandler
        material="color: red;"
        scale="20 20 20"
        gps-entity-place="latitude: <your-latitude>; longitude: <your-longitude>;"
      >
      </a-box>

      <a-camera gps-camera rotation-reader> </a-camera>
    </a-scene>
  </body>
</html>
```

You can use this exact approach for Image Tracking `a-nft` and Marker Based `a-entity` elements.
The `clickhandler` name can be customized, you can choose the one you like most, it's just a reference.

Keep in mind that this click/touch interaction is not handled by AR.js at all, it is all A-Frame based.
Always look on the A-Frame documentation for more details.

## Interaction with Overlayed DOM content

You can add interations by adding DOM HTML elements on the `body`.
For example, starting from this example:

```html
<!DOCTYPE html>
<html>
  <script src="https://aframe.io/releases/1.0.4/aframe.min.js"></script>
  <!-- we import arjs version without NFT but with marker + location based support -->
  <script src="https://raw.githack.com/AR-js-org/AR.js/master/aframe/build/aframe-ar.js"></script>
  <body style="margin : 0px; overflow: hidden;">
    <a-scene embedded arjs>
      <a-marker preset="hiro">
        <a-entity
          position="0 0 0"
          scale="0.05 0.05 0.05"
          gltf-model="https://arjs-cors-proxy.herokuapp.com/https://raw.githack.com/AR-js-org/AR.js/master/aframe/examples/image-tracking/nft/trex/scene.gltf"
        ></a-entity>
      </a-marker>
      <a-entity camera></a-entity>
    </a-scene>
  </body>
</html>
```

We can add on the body, outside the `a-scene`:

```html
<div class="buttons">
  <button class="say-hi-button"></button>
</div>
```

Then, we need to add some CSS to absolute positioning the DIV and BUTTON, and also some scripting to listen to click events.

You can customize your `a-scene` or content, like 3D models, play video, and so on. See on A-Frame Docs on how to change entity properties and work with events: https://aframe.io/docs/1.0.0/introduction/javascript-events-dom-apis.html.

We will end up with the following code:

```html
<!DOCTYPE html>
<html>
  <script src="https://aframe.io/releases/1.0.4/aframe.min.js"></script>
  <!-- we import arjs version without NFT but with marker + location based support -->
  <script src="https://raw.githack.com/AR-js-org/AR.js/master/aframe/build/aframe-ar.js"></script>
  <script>
    window.onload = function() {
      document
        .querySelector(".say-hi-button")
        .addEventListener("click", function() {
          // here you can change also a-scene or a-entity properties, like
          // changing your 3D model source, size, position and so on
          // or you can just open links, trigger actions...
          alert("Hi there!");
        });
    };
  </script>
  <style>
    .buttons {
      position: absolute;
      bottom: 0;
      left: 0;
      width: 100%;
      height: 5em;
      display: flex;
      justify-content: center;
      align-items: center;
      z-index: 10;
    }

    .say-hi-button {
      padding: 0.25em;
      border-radius: 4px;
      border: none;
      background: white;
      color: black;
      width: 4em;
      height: 2em;
    }
  </style>
  <body style="margin : 0px; overflow: hidden;">
    <div class="buttons">
      <button class="say-hi-button">SAY HI!</button>
    </div>
    <a-scene embedded arjs>
      <a-marker preset="hiro">
        <a-entity
          position="0 0 0"
          scale="0.05 0.05 0.05"
          gltf-model="https://arjs-cors-proxy.herokuapp.com/https://raw.githack.com/AR-js-org/AR.js/master/aframe/examples/image-tracking/nft/trex/scene.gltf"
        ></a-entity>
      </a-marker>
      <a-entity camera></a-entity>
    </a-scene>
  </body>
</html>
```

## Custom Events

AR.js dispatches several Custom Events.

Some of them are general, others are specific for AR Feature. Here's the full list.

| Custom Event name                 | Description                                                                         | Payload                                                                                   | Source File               | Feature                              |
| --------------------------------- | ----------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- | ------------------------- | ------------------------------------ |
| `arjs-video-loaded`               | Fired when camera video stream has been appended to the DOM                         | `{ detail: { component: <HTMLElement> }}`                                                 | threex-artoolkitsource.js | all                                  |
| `camera-error`                    | Fired when camera video stream could not be retrieved                               | `{ error: <Error> }`                                                                      | threex-artoolkitsource.js | all                                  |
| `camera-init`                     | Fired when camera video stream has been retrieved correctly                         | `{ stream: <MediaStream> }`                                                               | threex-artoolkitsource.js | all                                  |
| `markerFound`                     | Fired when a marker in Marker Based, or a picture in Image Tracking, has been found | -                                                                                         | component-anchor.js       | only Image Tracking and Marker Based |
| `markerLost`                      | Fired when a marker in Marker Based, or a picture in Image Tracking, has been lost  | -                                                                                         | component-anchor.js       | only Image Tracking and Marker Based |
| `arjs-nft-loaded`   | Fired when a nft marker is full loaded  |   | threex-armarkercontrols-nft-start.js   | only Image Tracking  |
| `gps-camera-update-positon`       | Fired when `gps-camera` has updated its position                                    | `{ detail:` `{ position: <GeolocationCoordinates>,` `origin: <GeolocationCoordinates> }}` | gps-camera.js             | only Location Based                  |
| `gps-entity-place-update-positon` | Fired when `gps-entity-place` has updated its position                              | `{ detail: { distance: <Number> }}`                                                       | gps-entity-place.js       | only Location Based                  |
| `gps-entity-place-added`          | Fired when the `gps-entity-place` has been added                                    | `{ detail: { component: <HTMLElement> }}`                                                 | gps-entity-place.js       | only Location Based                  |
| `gps-camera-origin-coord-set`     | Fired when the origin coordinates are set                                           | -                                                                                         | gps-camera.js             | only Location Based                  |
| `gps-entity-place-loaded`         | Fired when the `gps-entity-place` has been - see 'loaded' event of A-Frame entities | `{ detail: { component: <HTMLElement> }}`                                                 | gps-entity-place.js       | only Location Based                  |

### Internal Loading Events

⚡️ Both Image Tracking and Location Based automatically handle an internal event when

- origin location has been set
- Image Tracking (Image Descriptors) are fully loaded

And automatically remove from the DOM elements that match the `.arjs-loader` selector.

You can add any custom loader that will be remove in the above situations, just use the `.arjs-loader` class on it.
