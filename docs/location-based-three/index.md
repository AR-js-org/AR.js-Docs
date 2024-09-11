# Location-based AR.js with three.js - Develop a simple Points of Interest app

AR.js 3.4 features a pure three.js API for location-baed AR. Here is a series of tutorials taking you through how to use it, from the basics to a more advanced example: a simple but working Points of Interest app using a live web API.

It is expected that you have some basic [three.js](https://threejs.org) experience.

**Do note that this code will not work on Firefox on a mobile device due to limitations of the device orientation API; absolute orientation cannot be obtained. Chrome on Android is recommended.**

## Installing

It is recommended to install via `npm` and build with a bundler such as Webpack.
and `import` it into your application.

Here is a sample `package.json`:

```
{
    "dependencies": {
        "@ar-js-org/ar.js": "3.4.5"
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
This will build a bundle named `bundle.js` in the `dist` subdirectory from a source file `index.js`. This will be assumed in the examples.

- [Part 1: Hello World](part1.md)
- [Part 2: Using the GPS and Device Orientation](part2.md)
- [Part 3: Connecting to a web API](part3.md)

