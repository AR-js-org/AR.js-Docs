# Image Tracking

Image Tracking makes possible to scan a picture, drawing, any image, and show content over it.

All the following examples are with A-Frame, for semplicity. You can use three.js if you want. See on the official repository the [three.js examples](https://github.com/AR-js-org/AR.js/tree/master/three.js/examples), search for 'nft' example.

All A-Frame examples for Image Tracking can be found [here](https://github.com/AR-js-org/AR.js/tree/master/aframe/examples/image-tracking/).

## Getting started with Image Tracking

Thanks to [Daniel Fernandes](https://twitter.com/DanielCarnaux) for contribution on this docs section.

Natural Feature Tracking or NFT is a technology that enables the use of images instead of markers like QR Codes or the Hiro.

**Now you can use any image you want!**

The software tracks interesting points in the image and from that it estimates the position of the camera. These interesting points (aka "Image Descriptors") are created using the `NFT Marker Creator`, a tool available to you for creating NFT markers, it has two versions the [WEB APP](https://carnaux.github.io/NFT-Marker-Creator/)(recommended), and the [NODE APP](https://github.com/Carnaux/NFT-Marker-Creator). There is also a fork on the AR.js Github organisation, but as for now, [Daniel Fernandes](https://twitter.com/DanielCarnaux) version works perfectly.

## Choose good images

If you want to understand the creation of markers in more depth, check out the  NFT Marker Creator [wiki](https://github.com/Carnaux/NFT-Marker-Creator/wiki/Creating-good-markers).

## Create Image Descriptors

Once you have chosen your image, you can either use the [web version](https://carnaux.github.io/NFT-Marker-Creator/) or the node version.

If you're using the node version, this is the basic command to run:

```bash
node app.js -i <path-to-the-img/image-name.jpg/png>
```

And you will find the Image Descriptors files on the `output` folder. In the web version, the generator will automatically download the files from your browser.

In either cases, you will end up with three files as Image Descriptors, with `.fset`, `.fset3`, `.iset`.
Each of them will have the same prefix before the file extension. That one will be the Image Descriptor name that you will use on the AR.js web app.

## Render the content

Now it's time to create the actual Web app.



