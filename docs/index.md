# AR.js - Augmented Reality on the Web

<img src="./logo.png" height="200" />

---

AR.js is a lightweight library for Augmented Reality on the Web, coming with features like Image Tracking, Location based AR and Marker tracking.

## Why AR.js

We believe in the Web, a collaborative and accessible place. We also believe in the Augmented Reality technology, as a new communication medium, that can help people to see the reality in new, exciting ways. We see Augmented Reality (AR) used everyday for a lot of useful applications, from art, to education, also for fun.

We strongly believe that such a powerful technology, that can help people and leverage their creativity, should be free in some way. Also collaborative, if possible. And so, we continue the work started by Jerome Etienne, in bringing AR on the Web, as a free and Open Source technology.

Thank you for being interested in this, if you'd like to collaborate in any way, contact us ([https://twitter.com/nicolocarp](https://twitter.com/nicolocarp)).

The project is now under a Github organization, that you can find at [https://github.com/ar-js-org](https://github.com/ar-js-org) and you can ask to be part of it, for free.

### AR types

Ar.js features the following types of Augmented Reality, on the Web:

- Image Tracking
- Location Based AR
- Marker Tracking.

### Key points

- **Very Fast** : It runs efficiently even on phones
- **Web-based** : It is a pure web solution, so no installation required. Full javascript based on three.js + A-Frame + jsartoolkit5
- **Open Source** : It is completely open source and free of charge!
- **Standards** : It works on any phone with [webgl](http://caniuse.com/#feat=webgl) and [webrtc](http://caniuse.com/#feat=stream)

AR.js has reached version 3. This is the official repository: [https://github.com/AR-js-org/AR.js](https://github.com/jeromeetienne/AR.js/issues).
If you want to visit the old AR.js repository, here it is: [https://github.com/jeromeetienne/AR.js](https://github.com/jeromeetienne/AR.js).

## Import the library

AR.js from version 3 has a new structure.

**AR.js is coming in two, different build. They are both maintained. They are exclusive.**

The file you want to import depends on what features you want, and also which render library you want to use (A-Frame or three.js).

AR.js uses [jsartoolkit5](https://github.com/artoolkitx/jsartoolkit5) for tracking, but can display augmented content with either [three.js](https://threejs.org/) or [A-Frame](https://aframe.io/).

You can import AR.js in one version of your choice, using the `<script>` tag on your HTML.

---

**AR.js with Image Tracking + Location Based AR**

Import AFRAME version:

```html
<script src="https://raw.githack.com/AR-js-org/AR.js/master/aframe/build/aframe-ar-nft.js">
```

Import three.js version:

```html
<script src="https://raw.githack.com/AR-js-org/AR.js/master/three.js/build/ar-nft.js">
```

---

**AR.js with Marker Tracking + Location Based AR:**

Import AFRAME version:

```html
<script src="https://raw.githack.com/AR-js-org/AR.js/master/aframe/build/aframe-ar.js">
```

Import three.js version:

```html
<script src="https://raw.githack.com/AR-js-org/AR.js/master/three.js/build/ar.js">
```

## Requirements

Some requirements and known restrictions are listed below:

- It works on every phone with [webgl](http://caniuse.com/#feat=webgl) and [webrtc](http://caniuse.com/#feat=stream).
- Marker based is very lightweight, while Image Tracking is more CPU consuming
- You cannot use Chrome on iOS, as Chrome on iOS did not support, at the moment, camera access
- On device with multi-cameras, Chrome may have problems on detecting the right one. Please use Firefox if you find that AR.js opens on the wrong camera. There is an open issue for this.
- To work with Location Based feature, your phone needs to have GPS sensors. Please, read carefully all suggestions that AR.js app popp-up for Location Based on iOS, as iOS requires user actions to activate geoposition.
- Location Based feature is only available on A-Frame
- Location Based can work with A-Frame up to version 0.9.2. Compatibility with A-Frame 1.0.0 and later is work in progress.

### Always deploy under https

Accessing to the phone camera or to camera GPS sensors, due to major browsers restrictions, can be done only under https websites.

All the examples you will see, and all AR.js web apps in general, have to be run on a server. You can use local server or deploy the static web app on the web.

Please, always run your examples on secure connections servers or localhost. [Github Pages](https://pages.github.com/) is a great way to have free and live websites under https.

## Getting started

Here we present three, basic examples, one for each AR feature. For specific documentation, on the top menu you can find every section, or you can click on the following links:

- [Image Tracking Documentation](./image-tracking.md)
- [Location Based Documentation](./location-based.md)
- [Marker Based Documentation](./marker-based.md)

### Image Tracking Example

Please follow this simple steps:

- Create a new project with the code below (or [**try this codepen**](https://codepen.io/nicolocarpignoli/pen/vYOeYKd) and go directly to the last step)
- Run it on a server
- Open the website on your phone
- Scan [this picture](https://raw.githack.com/AR-js-org/AR.js/master/aframe/examples/image-tracking/nft/trex-image.jpg) to see content through the camera.

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
    <!-- we use cors-anywhere proxy to avoid cross-origin problems -->
    <a-nft
      type="nft"
      url="https://arjs-cors-proxy.herokuapp.com/https://raw.githack.com/AR-js-org/AR.js/master/aframe/examples/image-tracking/nft/trex/trex-image/trex"
      smooth="true"
      smoothCount="10"
      smoothTolerance=".01"
      smoothThreshold="5"
    >
      <a-entity
        gltf-model="https://arjs-cors-proxy.herokuapp.com/https://raw.githack.com/AR-js-org/AR.js/master/aframe/examples/image-tracking/nft/trex/scene.gltf"
        scale="5 5 5"
        position="50 150 0"
      >
      </a-entity>
    </a-nft>
    <a-entity camera></a-entity>
  </a-scene>
</body>
```

### Location Based Example

Please follow this simple steps:

- Create a new project with the following snippet, and change `add-your-latitude` and `add-your-longitude` with your latitude and longitude, without the `<>`.
- Run it on a server
- Activate GPS on your phone and navigate to the example URL
- Look around. You should see the text looking at you, appearing in the requested position, even if you look around and move.

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <title>GeoAR.js demo</title>
    <!-- with location based, use aframe v0.9.2 -->
    <script src="https://aframe.io/releases/0.9.2/aframe.min.js"></script>
    <script src="https://unpkg.com/aframe-look-at-component@0.8.0/dist/aframe-look-at-component.min.js"></script>
    <script src="https://raw.githack.com/AR-js-org/AR.js/master/aframe/build/aframe-ar-nft.js"></script>
  </head>

  <body style="margin: 0; overflow: hidden;">
    <a-scene
      vr-mode-ui="enabled: false"
      embedded
      arjs="sourceType: webcam; debugUIEnabled: false;"
    >
      <a-text
        value="This content will always face you."
        look-at="[gps-camera]"
        scale="120 120 120"
        gps-entity-place="latitude: <add-your-latitude>; longitude: <add-your-longitude>;"
      ></a-text>
      <a-camera gps-camera rotation-reader> </a-camera>
    </a-scene>
  </body>
</html>
```

### Marker Based Example

Please follow this simple steps:

- Create a new project with the code below (or [**try this codepen**](https://codepen.io/nicolocarpignoli/pen/vMBgob) and go directly to the last step)
- Run it on a server
- Open the website on your phone
- Scan [this picture](https://raw.githack.com/AR-js-org/AR.js/master/aframe/examples/image-tracking/nft/trex-image.jpg) to see content through the camera.

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

### Advanced stuff

AR.js offers two ways, with A-Frame, to interact with the web page: clicks/touches events and Overlayed DOM interaction.

Also, there are several Custom Events triggered during the life cycle of an AR.js web app.

You can learn more about these aspects on the [UI and Events section](./ui-events.md).

### AR.js architecture

AR.js uses [jsartoolkit5](https://github.com/artoolkitx/jsartoolkit5) for tracking, but can display augmented content with either [three.js](https://threejs.org/) or [A-Frame](https://aframe.io/).

`three.js` folder contains

- source code for AR.js core, Marker based and Image Tracking
- examples for AR.js three.js based
- build for three.js AR.js based
- vendor stuff (jsartoolkit5)
- workers (used for Image Tracking).

When you find files that ends with `-nft` suffix, they're boundled only with the Image Tracking version.

A-Frame version of AR.js uses three.js parts as its core. A-Frame here is it simply a wrapper to write AR with Custom Components in HTML.

`aframe` folder contains

- source code for AR.js A-Frame (aka wrappers for Marker Based, Image Tracking components)
- source code for Location Based
- build for A-Frame AR.js based
- examples for A-Frame AR.js.
