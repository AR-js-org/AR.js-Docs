# UI and Custom Events

To make AR.js based Web App looking better and add UI capabilities, it's possible to treat is as common website.
Here you will learn how to use Raycaster, Custom Events and Interaction with overlayed DOM elements.

First of all, an introduction about click interactions: it is possible to click directly on content using A-Frame, we tested and had good result with version `0.9.2`. Keep in mind that clicks (or touch interactions) are not very reliable: it can work if you

- need only few clicks, not very reliable
- have to click on very few entities on screen
- clicks work only on the center of the screen, not at the angles, so beware that your entities, to be clicked, should be great enough and not positioned on angles.

What I suggest is that you have always to remember that AR.js is Web AR, that means, you can use **any** DOM elements interactions you want.
They work perfectly and most of the times, the Overlayed DOM content interaction is enough for common needs.

## Handle clicks on AR content

We can use click/touch interactions through raycaster, following [A-Frame Docs](https://aframe.io/docs/1.0.0/introduction/interactions-and-controllers.html).

We can register an event listener for any a-frame entity. So this method works also for Marker Based, Image Tracking and Location Based.
We will show it on a Location Based examples, registering the click on `a-entity` - so the same can be done with `a-marker` or `a-nft`.

## Interaction with Overlayed DOM content

You can add interations by adding DOM HTML elements on the `body`.
For example, startinf from this example:

```html
<!DOCTYPE html>
<html>
  <script src="https://aframe.io/releases/1.0.0/aframe.min.js"></script>
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
  <script src="https://aframe.io/releases/1.0.0/aframe.min.js"></script>
  <!-- we import arjs version without NFT but with marker + location based support -->
  <script src="https://raw.githack.com/AR-js-org/AR.js/master/aframe/build/aframe-ar.js"></script>
  <script>
    window.onload = function() {
        document.querySelector('.say-hi-button').addEventListener('click', function() {
            // here you can change also a-scene or a-entity properties, like
            // changing your 3D model source, size, position and so on
            // or you can just open links, trigger actions...
            alert('Hi there!');
        })
    }
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

## Custom Events list

AR.js triggers several Custom Events. Some of them are general, other are specific for AR Features. Here's the full list.


