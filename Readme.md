# Web assembly ImageMagick [![Build Status](https://dev.azure.com/oneeyedelf1/wasm-imagemagick/_apis/build/status/KnicKnic.WASM-ImageMagick)](https://dev.azure.com/oneeyedelf1/wasm-imagemagick/_build/latest?definitionId=1)
This project is not affiliated with [ImageMagick](https://www.imagemagick.org) , but is merely recompiling the code to be [WebAssembly](https://webassembly.org/). I did this because I want to bring the power of ImageMagick to the browser.


## Demos and examples

 * Basic playground (React & TypeScript project): [![Basic playground (React & TypeScript project)](https://codesandbox.io/static/img/play-codesandbox.svg)](https://codesandbox.io/s/lp7lxz6l59).

 * Image Diff Example (React & TypeScript project): [![Basic playground for image diff (React & TypeScript project)](https://codesandbox.io/static/img/play-codesandbox.svg)](https://codesandbox.io/s/yvn6rkr16z).

 * [Playground with several transformation examples and image formats](https://cancerberosgx.github.io/autumn-leaves/#/convertDemo). It also shows the output of transformations made with ImageMaick in the browser to verify wasm-imagemagick output the right thing.  

 * [Picture Frame editor](https://cancerberosgx.github.io/autumn-leaves/#/imageFrame).

 * Simple example. See [samples/rotate#code](samples/rotate#code). A simple webpage that has image in array and loads magickApi.js to rotate file. Demonstration site [https://knicknic.github.io/imagemagick/rotate/](https://knicknic.github.io/imagemagick/rotate/)

 * [https://knicknic.github.io/imagemagick/](https://knicknic.github.io/imagemagick/) a commandline sample of using ImageMagick
    * For code see [samples/cmdline](samples/cmdline)

 * Used in [Croppy](https://knicknic.github.io/croppy/) to split webcomics from one long vertical strip into many panels.
    * For code see https://github.com/KnicKnic/croppy.

## Status

### Image formats supported

Supports PNG, TIFF, JPEG, + Native ImageMagick such as BMP, GIF, [PhotoShop](https://www.adobe.com/products/photoshop.html), [GIMP](https://www.gimp.org/)

### Features **not** supported 

 * [Text](https://www.imagemagick.org/Usage/text/)
 * [Fourier Transforms](https://www.imagemagick.org/Usage/fourier/)


## Usage with npm

```sh
npm install --save wasm-imagemagick
```

**IMPORTANT:  

**Don't forget to copy `magick.wasm` and `magick.js`** files to the folder where your `index.html` is being served:

```sh
cp node_modules/wasm-imagemagick/dist/magick.wasm .
cp node_modules/wasm-imagemagick/dist/magick.js .
```

### High level API and utilities

`wasm-imagemagick` comes with some easy to use APIs for creating image files from urls, executing multiple commands reusing output images and nicer command syntax, and utilities to handle files, html images, input elements, image comparison, metadata extraction, etc. The following example is equivalent to the previous using these APIs: 

```ts
import { buildInputFile, execute, loadImageElement } from 'wasm-imagemagick'

const {outputFiles} = await execute({
  inputFiles: [await buildInputFile('http://some-cdn.com/foo/fn.png', 'image1.png')],
  commands: [
    'convert image1.png -rotate 70 image2.gif',
    // heads up: the next command uses 'image2.gif' which was the output of previous command:
    'convert image2.gif -scale 23% image3.jpg',
  ],
})
await loadImageElement(outputFiles[0], document.getElementById('outputImage'))
```

### Accessing stdout, stderr, exitCode

This other example executes `identify` command to extract information about an image. As you can see, we access `stdout` from the execution result and check for errors using `exitCode` and `stderr`: 

```ts
import { buildInputFile, execute } from 'wasm-imagemagick'

const { stdout, stderr, exitCode } = await execute({
    inputFiles: [await buildInputFile('foo.gif')], 
    commands: `identify foo.gif`
})
if(exitCode === 0) 
    console.log('foo.gif identify output: ' + stdout.join('\n'))
else 
    console.error('foo.gif identify command failed: ' + stderr.join('\n'))
```

### low-level example

As demonstration purposes, the following example doesn't use any helper provided by the library, only the `call()` function which only accept one command, in array syntax only:

```js
import { call } from 'wasm-imagemagick'

// build an input file by fetching its content
const fetchedSourceImage = await fetch("assets/rotate.png")
const content = new Uint8Array(await fetchedSourceImage.arrayBuffer());
const image = { name: 'srcFile.png', content }

const command = ["convert", "srcFile.png", '-rotate', '90', '-resize', '200%', 'out.png']
const result = await call([image], command)

// response can be multiple files (example split) here we know we just have one
const firstOutputImage = result.processedFiles[0]

// render the output image into an existing <img> element
const outputImage = document.getElementById('outputImage')
outputImage.src = URL.createObjectURL(firstOutputImage.blob)
```


## Loading directly from html file

If you are not working in a npm development environment you can still load the library bundle .js file. It supports being imported as JavaScript standard module or as a UMD module. Again, don't forget to copy `magick.js`, `magick.wasm` in the same folder as your html file.:


### Importing it as JavaScript standard module: 

```html
<script type="module">
    import { execute, loadImageElement, buildInputFile } from '../../dist/bundles/wasm-imagemagick.esm-es2018.js'
    // ... same snippet as before
</script>
```

[See working example](https://github.com/KnicKnic/WASM-ImageMagick/blob/master/tests/bundles/esm-test.html)


### Using the UMD bundle in AMD projects (requirejs)

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/require.js/2.3.6/require.min.js"></script>
<script src="../../dist/bundles/wasm-imagemagick.umd-es5.js"></script>
<script>
require(['wasm-imagemagick'], function (WasmImagemagick) {
    const { execute, loadImageElement, buildInputFile } = WasmImagemagick
    // ... same snippet as before
```

[See working example](https://github.com/KnicKnic/WASM-ImageMagick/blob/master/tests/bundles/umd-test-requirejs.html)


### Using the UMD bundle without libraries

```html
<script src="../../dist/bundles/wasm-imagemagick.umd-es5.js"></script>
<script>
    const { execute, loadImageElement, buildInputFile } = window['wasm-imagemagick']
    // ... same snippet as before
```

[See working example](https://github.com/KnicKnic/WASM-ImageMagick/blob/master/tests/bundles/umd-test-nolibrary.html)


## API reference documentation

[API Reference Documentation](./apidocs)


## Build instructions

```sh
git clone --recurse-submodules https://github.com/KnicKnic/WASM-ImageMagick.git

cd WASM-ImageMagick

docker build -t wasm-imagemagick-build-tools .

docker run --rm -it --workdir /code -v "$PWD":/code wasm-imagemagick-build-tools bash ./build.sh

#windows cmd
#docker run --rm -it --workdir /code -v %CD%:/code wasm-imagemagick-build-tools bash ./build.sh
```

Produces `magick.js` & `magick.wasm` in the current folder.

Note: `npm run build` will perform all the previous commands plus compiling the TypeScript project.


## Run tests

`npm test` will run some tests with nodejs located at `./tests/rotate`.

`npm run test-browser` will run spec in a headless chrome browser. This tests are located at `./spec/`. `npm run test-browser-server` will serve the test so you can debug them with a browser. 