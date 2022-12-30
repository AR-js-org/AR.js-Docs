# UI and Custom Events

To make AR.js based Web App look better and add UI capabilities, it's possible to treat is as common website.
Here you will learn how to use Raycaster, Custom Events and Interaction with overlayed DOM elements.

## Handle clicks on AR content

<img src='https://miro.medium.com/max/1200/1*7aR8--E_lqt91Ziw1VCCTw.gif' />

It's now possible to use AR.js (marker based or image tracking) with a-frame latest versions (1.0.0 and above) in order
to have touch gestures to zoom and rotate your content! Disclaimer: this will work for your entire `a-scene`, so it's not a real
option if you have to handle different interactions for multiple markers. It will work like charm if you have one marker/image for scene.

Check Fabio Cortès [great walkthrough](https://medium.com/@fabiojcortes/manipulate-your-3d-content-with-gestures-in-ar-js-78da4c076607) in order to add this feature on your AR.js web app.

You can use this exact approach for Image Tracking `a-nft` and Marker Based `a-entity` elements.
The `clickhandler` name can be customized, you can choose the one you like most, it's just a reference.

Keep in mind that this click/touch interaction is not handled by AR.js at all, it is all A-Frame based.
Always look on the A-Frame documentation for more details.

[Check out the tutorial](https://medium.com/@fabiojcortes/manipulate-your-3d-content-with-gestures-in-ar-js-78da4c076607)

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
        <!-- we use cors proxy to avoid cross-origin problems -->

        <!--
          ⚠️⚠️⚠️
          https://arjs-cors-proxy.herokuapp.com/ is now offline, Heroku has dismissed all his free plans from November 2022.
          You need to host your own proxy and use it instead. The proxy is based on CORS Anywhere (see https://github.com/Rob--W/cors-anywhere).
          ⚠️⚠️⚠️
        -->
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
    window.onload = function () {
      document
        .querySelector(".say-hi-button")
        .addEventListener("click", function () {
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

## Custom Events

AR.js dispatches several Custom Events.

Some of them are general, others are specific for AR Feature. Here's the full list.

| Custom Event name                 | Description                                                                         | Payload                                                                                   | Source File                          | Feature                              |
| --------------------------------- | ----------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- | ------------------------------------ | ------------------------------------ |
| `arjs-video-loaded`               | Fired when camera video stream has been appended to the DOM                         | `{ detail: { component: <HTMLElement> }}`                                                 | threex-artoolkitsource.js            | all                                  |
| `camera-error`                    | Fired when camera video stream could not be retrieved                               | `{ error: <Error> }`                                                                      | threex-artoolkitsource.js            | all                                  |
| `camera-init`                     | Fired when camera video stream has been retrieved correctly                         | `{ stream: <MediaStream> }`                                                               | threex-artoolkitsource.js            | all                                  |
| `markerFound`                     | Fired when a marker in Marker Based, or a picture in Image Tracking, has been found | -                                                                                         | component-anchor.js                  | only Image Tracking and Marker Based |
| `markerLost`                      | Fired when a marker in Marker Based, or a picture in Image Tracking, has been lost  | -                                                                                         | component-anchor.js                  | only Image Tracking and Marker Based |
| `arjs-nft-loaded`                 | Fired when a nft marker is full loaded                                              |                                                                                           | threex-armarkercontrols-nft-start.js | only Image Tracking                  |
| `gps-camera-update-positon`       | Fired when `gps-camera` has updated its position                                    | `{ detail:` `{ position: <GeolocationCoordinates>,` `origin: <GeolocationCoordinates> }}` | gps-camera.js                        | only Location Based                  |
| `gps-entity-place-update-positon` | Fired when `gps-entity-place` has updated its position                              | `{ detail: { distance: <Number> }}`                                                       | gps-entity-place.js                  | only classic and projected Location Based                  |
| `gps-entity-place-added`          | Fired when the `gps-entity-place` has been added                                    | `{ detail: { component: <HTMLElement> }}`                                                 | gps-entity-place.js                  | only classic and projected Location Based                   |
| `gps-camera-origin-coord-set`     | Fired when the origin coordinates are set                                           | -                                                                                         | gps-camera.js                        | only classic and projected Location Based                  |
| `gps-entity-place-loaded`         | Fired when the `gps-entity-place` has been - see 'loaded' event of A-Frame entities | `{ detail: { component: <HTMLElement> }}`                                                 | gps-entity-place.js                  | only classic and projected Location Based                  |

### Internal Loading Events

⚡️ Both Image Tracking and Location Based automatically handle an internal event when

- origin location has been set
- Image Tracking (Image Descriptors) are fully loaded

And automatically remove from the DOM elements that match the `.arjs-loader` selector.

You can add any custom loader that will be remove in the above situations, just use the `.arjs-loader` class on it.

## Trigger actions when image has been found

You can trigger any action you want when marker/image has been found. You can avoid linking a content to a marker/image and
only trigger an action (like a redirect to an external website) when the anchor has been found by the camera.

```html
<script src="https://cdn.jsdelivr.net/gh/aframevr/aframe@1c2407b26c61958baa93967b5412487cd94b290b/dist/aframe-master.min.js"></script>
<script src="https://raw.githack.com/AR-js-org/AR.js/master/aframe/build/aframe-ar-nft.js"></script>

<style>
  .arjs-loader {
    height: 100%;
    width: 100%;
    position: absolute;
    top: 0;
    left: 0;
    background-color: rgba(0, 0, 0, 0.8);
    z-index: 9999;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .arjs-loader div {
    text-align: center;
    font-size: 1.25em;
    color: white;
  }
</style>
<script>
  AFRAME.registerComponent('markerhandler', {
    init: function () {
      this.el.sceneEl.addEventListener('markerFound', () => {
        // redirect to custom URL
        window.location = 'https://github.com/AR-js-org/AR.js';
      });
  });
  },
</script>

<body style="margin : 0px; overflow: hidden;">
  <!-- minimal loader shown until image descriptors are loaded -->
  <div class="arjs-loader">
    <div>Loading, please wait...</div>
  </div>
  <a-scene
    vr-mode-ui="enabled: false;"
    renderer="logarithmicDepthBuffer: true;"
    embedded
    arjs="trackingMethod: best; sourceType: webcam;debugUIEnabled: false;"
  >
    <!-- we use cors proxy to avoid cross-origin problems -->
    <!-- we use the trex image shown on the homepage of the docs -->
    <a-nft
      markerhandler
      type="nft"
      url="https://arjs-cors-proxy.herokuapp.com/https://raw.githack.com/AR-js-org/AR.js/master/aframe/examples/image-tracking/nft/trex/trex-image/trex"
    >
    </a-nft>
    <a-entity camera></a-entity>
  </a-scene>
</body>
```

## Trigger action when marker has been found

```html
<script src="https://cdn.jsdelivr.net/gh/aframevr/aframe@1c2407b26c61958baa93967b5412487cd94b290b/dist/aframe-master.min.js"></script>
<script src="https://raw.githack.com/AR-js-org/AR.js/master/aframe/build/aframe-ar-nft.js"></script>

<script>
  AFRAME.registerComponent('markerhandler', {
    init: function () {
      this.el.sceneEl.addEventListener('markerFound', () => {
        // redirect to custom URL e.g. google.com
        window.location = 'https://www.google.com/';
      })
    }
  });
</script>

<body style="margin : 0px; overflow: hidden;">
  <a-scene
    vr-mode-ui="enabled: false;"
    renderer="logarithmicDepthBuffer: true;"
    embedded
    arjs="trackingMethod: best; sourceType: webcam; debugUIEnabled: false; detectionMode: mono_and_matrix; matrixCodeType: 3x3;">

    <a-marker markerhandler type='barcode' value='7'>
        <a-box position='0 0.5 0' color="yellow"></a-box>
    </a-marker>

    <a-entity camera></a-entity>
  </a-scene>
</body>
```

## Get distance from marker

```
// import this on your HTML
window.addEventListener('load', () => {
    const camera = document.querySelector('[camera]');
    const marker = document.querySelector('a-marker');
    let check;

    marker.addEventListener('markerFound', () => {
        let cameraPosition = camera.object3D.position;
        let markerPosition = marker.object3D.position;
        let distance = cameraPosition.distanceTo(markerPosition)

        check = setInterval(() => {
            cameraPosition = camera.object3D.position;
            markerPosition = marker.object3D.position;
            distance = cameraPosition.distanceTo(markerPosition)

            // do what you want with the distance:
            console.log(distance);
        }, 100);
    });

    marker.addEventListener('markerLost', () => {
      clearInterval(check);
    })
})
```
