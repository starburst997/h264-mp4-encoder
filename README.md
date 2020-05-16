# Introducing h264-mp4-encoder
Encode directly to H264 and ouput as an MP4 in node or on the web with WebAssembly! Works with the HTML5 Canvas :)

[![Build Status](https://travis-ci.org/TrevorSundberg/h264-mp4-encoder.svg?branch=master)](https://travis-ci.org/TrevorSundberg/h264-mp4-encoder)
```
npm install h264-mp4-encoder
```

# Example
Web:
```html
<script src="https://unpkg.com/h264-mp4-encoder/embuild/dist/h264-mp4-encoder.web.js"></script>
```
JavaScript:
```js
const HME = require("h264-mp4-encoder");
```
TypeScript:
```ts
import * as HME from "h264-mp4-encoder";
```

Example:
```js
HME.createH264MP4Encoder().then(encoder => {
    encoder.width = 100;
    encoder.height = 100;
    encoder.initialize();
    // Add a single gray frame, the alpha is ignored.
    encoder.addFrameRgba(new Uint8Array(encoder.width * encoder.height * 4).fill(128))
    encoder.finalize();
    const uint8Array = encoder.FS.readFile("output.mp4");
    console.log(uint8Array);
    encoder.delete();
})
````

You can also use `await`:
```js
const encoder = await HME.createH264MP4Encoder();
```

# API

```ts
/** Construct the H264MP4Encoder. Waits for the WASM to complete loading before returning. */
async function createH264MP4Encoder(): Promise<H264MP4Encoder>;

interface H264MP4Encoder {
    /**
     * Name of the file that we output.
     * After `encoder.finalize()` use `encoder.FS.readFile(encoder.outputFilename)` after `finalize`.
     */
    outputFilename: number;

    /** Width of the input frames and output video. Must be a multiple of 2. */
    width: number;

    /** Height of the input frames and output video. Must be a multiple of 2. */
    height: number;

    /** Frame rate of the output video. */
    frameRate: number;

    /** The bitrate in kbps relative to the frame_rate. Overwrites quantization_parameter. */
    kbps: number;

    /** Speed where 0 means best quality and 10 means fastest speed [0..10]. */
    speed: number;

    /** Higher means better compression, and lower means better quality [10..51]. */
    quantizationParameter: number;

    /** Key frame period. */
    groupOfPictures: number;

    /** Use temporal noise supression. */
    temporalDenoise: number;

    /** Each NAL unit will be approximately capped at this size (0 means unlimited). */
    desiredNaluBytes: number;

    /** Prints extra debug information. */
    debug: number;

    /** Initialize the encoder. After calling this, all parameters above will be readonly. */
    initialize(): void;

    /** Add a frame in YUV format (expected size: width * height * 3 / 2). */
    addFrameYuv(buffer: ArrayBuffer | Uint8Array | Uint8ClampedArray | Int8Array | string): void;

    /**
     * Add a frame in RGBA format (expected size: width * height * 4).
     * Alpha is ignored but is kept for convenience of working with the HTML5 canvas.getImageData().
     */
    addFrameRgba(buffer: ArrayBuffer | Uint8Array | Uint8ClampedArray | Int8Array | string): void;

    /** Finish outputting the video to the `outputFilename`. */
    finalize(): void;

    /** Delete this object and free all resources. Should not be used again. */
    delete(): void;

    /**
     * A reference to Emscripten's virtual file system.
     * The output files will be written here under the provided `outputFilename`.
     * This file system is *shared* between all H264MP4Encoders.
     */
    FS: typeof FS;
}
```

# How?
This library brings together the public domain [minih264](https://github.com/lieff/minih264) H264 encoder, as well as the MPL 1.1 licenced [libmp4v2](https://github.com/sergiomb2/libmp4v2) to write the H264 NAL unit data into an MP4 file. Both of these are compiled in Emscripten within a Docker image and Webpacked to create easily importable modules. This repo uses submodules that are forks of those two libraries with changes for Emscripten (and to comply with the MPL 1.1 license).

# Where is the WebAssembly?
To make packaging considerably easier, I built with Emscripten's `-s SINGLE_FILE=1` which base64 encodes the wasm inside the JavaScript file. You're welcome to remove this option to get a wasm file, but you'll need to handle Webpack importing as well as the `locateFile` within the Emscripten Module object.

# Building on Ubuntu
You must have docker installed, however all other depdendencies for building for node/web are within the Docker container).

```bash
./build.sh
```

To run the C++ code natively (not using Emscripten) you must have cmake and a compiler:
```bash
git submodule update --init --recursive
mkdir build
cd build
cmake ..
cmake --build --parallel .
```