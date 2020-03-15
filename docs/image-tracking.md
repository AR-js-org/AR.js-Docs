# Image Tracking

Image Tracking makes possible to scan a picture, a drawing, any image, and show content over it.

All the following examples are with A-Frame, for semplicity. You can use three.js if you want. See on the official repository the [nft three.js example](https://github.com/AR-js-org/AR.js/blob/master/three.js/examples/nft.html).

All A-Frame examples for Image Tracking can be found [here](https://github.com/AR-js-org/AR.js/tree/master/aframe/examples/image-tracking/).

## Getting started with Image Tracking

Natural Feature Tracking or NFT is a technology that enables the use of images instead of markers like QR Codes or the Hiro marker.

The software tracks interesting points in the image and using them, it estimates the position of the camera. These interesting points (aka "Image Descriptors") are created using the `NFT Marker Creator`, a tool available for creating NFT markers. It comes in two versions: the [Web version](https://carnaux.github.io/NFT-Marker-Creator/)(recommended), and the [node.js version](https://github.com/Carnaux/NFT-Marker-Creator). There is also a fork of this project on the AR.js Github organisation, but as for now, [Daniel Fernandes](https://twitter.com/DanielCarnaux) version works perfectly.

*Thanks to [Daniel Fernandes](https://twitter.com/DanielCarnaux) for contribution on this docs section.*

## Choose good images

If you want to understand the creation of markers in more depth, check out the  NFT Marker Creator [wiki](https://github.com/Carnaux/NFT-Marker-Creator/wiki/Creating-good-markers). It explains also why certain images work way better than others. An important factor is the DPI of the image: a good dpi (300 or more) will give a very good stabilization, while low DPI (like 72) will require the user to stay very still and close to the image, otherwise tracking will lag.

## Create Image Descriptors

Once you have chosen your image, you can either use the NFT Marker Creator in its [Web version](https://carnaux.github.io/NFT-Marker-Creator/) or the node version.

If you're using the node version, this is the basic command to run:

```bash
node app.js -i <path-to-the-img/image-name.jpg/png>
```

After that, you will find the Image Descriptors files on the `output` folder. In the web version, the generator will automatically download the files from your browser.

In either cases, you will end up with three files as Image Descriptors, with `.fset`, `.fset3`, `.iset`.
Each of them will have the same prefix before the file extension. That one will be the Image Descriptor name that you will use on the AR.js web app.
For example: with files `trex.fset`, `trex.fset3` and `trex.iset`, your Image Descriptors name will be `trex`.

## Render the content

Now it's time to create the actual AR web app.

```html
<!-- import aframe and then ar.js with image tracking / location based features -->
<script src="https://cdn.jsdelivr.net/gh/aframevr/aframe@1c2407b26c61958baa93967b5412487cd94b290b/dist/aframe-master.min.js"></script>
<script src="https://raw.githack.com/AR-js-org/AR.js/master/aframe/build/aframe-ar-nft.js"></script>

<!-- style for the loader -->
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
  <!-- minimal loader shown until image descriptors are loaded. Loading may take a while according to the device computational power -->
  <div class="arjs-loader">
    <div>Loading, please wait...</div>
  </div>

  <!-- a-frame scene -->
  <a-scene
    vr-mode-ui="enabled: false;"
    renderer="logarithmicDepthBuffer: true;"
    embedded
    arjs="trackingMethod: best; sourceType: webcam;debugUIEnabled: false;"
  >
    <!-- a-nft is the anchor that defines an Image Tracking entity -->
    <!-- on 'url' use the path to the Image Descriptors created before. -->
    <!-- the path should end with the name without the extension e.g. if file is 'pinball.fset' the path should end with 'pinball' -->
    <a-nft
      type="nft"
      url="<path-to-your-image-descriptors>"
      smooth="true"
      smoothCount="10"
      smoothTolerance=".01"
      smoothThreshold="5"
    >
        <!-- as a child of the a-nft entity, you can define the content to show. here's a GLTF model entity -->
        <a-entity
            gltf-model="<path-to-your-model>"
            scale="5 5 5"
            position="50 150 0"
        >
        </a-entity>
    </a-nft>
    <!-- static camera that moves according to the device movemenents -->
    <a-entity camera></a-entity>
  </a-scene>
</body>
```

See on the comments above, inline on the code, for explanations.

You can refer to [A-Frame docs](https://aframe.io/docs/1.0.0/introduction/) to know everything about content and customization. You can add geometries, 3D models, videos, images. And you can customize their position, scale, rotation and so on.

The only custom component here is the `a-nft`, the Image Tracking HTML anchor.

### `<a-nft\>`

Here are the attributes for this entity

| Attribute | Description | Component Mapping |
| --- | --- | --- |
| type | type of marker - ['nft' only valid value] | artoolkitmarker.type |
| url | url of the Image Descriptors, without extension | artoolkitmarker.descriptorsUrl |
| emitevents | emits 'markerFound' and 'markerLost' events - ['true', 'false'] | - |
| smooth | turn on/off camera smoothing - ['true', 'false'] - default: false | - |
| smoothCount | number of matrices to smooth tracking over, more = smoother but slower follow - default: 5 | - |
| smoothTolerance | distance tolerance for smoothing, if smoothThreshold # of matrices are under tolerance, tracking will stay still - default: 0.01 | - |
| smoothThreshold | threshold for smoothing, will keep still unless enough matrices are over tolerance - default: 2 | - |
| size | size of the marker in meter | artoolkitmarker.size |

⚡️ It is suggested to use `smooth`, `smoothCount` and `smoothTolerance` because of weak stabilization of content in Image Tracking. Thanks to smoothing, content is way more stable, from 3D models to 2D videos.
