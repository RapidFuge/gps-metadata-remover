# `@xoi/gps-metadata-remover`
> Frontend-friendly Javascript package that removes GPS metadata from images and videos

> This is a fork of [@xoi/gps-metadata-remover](https://www.npmjs.com/package/@xoi/gps-metadata-remover). I recommend using the original version. This version only changes the default LogLevel from debug to none.

Takes a photo or video file and writes over in-place where GPS metadata is detected. Removes
Exif from images and various forms of GPS metadata from videos.

Currently compatible with JPG, PNG, TIF, MOV, MP4

[![npm version](https://badge.fury.io/js/%40xoi%2Fgps-metadata-remover.svg)](https://badge.fury.io/js/%40xoi%2Fgps-metadata-remover)

## Installation

`yarn add @xoi/gps-metadata-remover` or `npm i @xoi/gps-metadata-remover`

## Usage

`removeLocation` is the main removal function, although some other utility functions
are provided in the index to help read in data correctly if needed. This is how the package
looks being used in react native with react-native-fs:

```javascript
import { removeLocation, base64StringToArrayBuffer } from '@xoi/gps-metadata-remover'

const read = async (size, offset) => {
  const base64Data = await rnfs.read(destPath, size, offset, 'base64')
  return base64StringToArrayBuffer(base64Data)
}
const write = async (writeValue, entryOffset, encoding) => {
  await rnfs.write(destPath, writeValue, entryOffset, encoding)
}
const gpsWasRemoved = await removeLocation(destPath, read, write)
```

`removeLocation` returns `true` if GPS metadata was found and rewritten and `false`
if no GPS metadata was found and nothing was rewritten.

This package is platform-agnostic, so the client is expected to pass in filesystem
`read` and `write` functions that work for their platform. The functions should match
the following types:

```javascript
export type ReadFunction = (size: number, offset: number) => Promise<Buffer>
```
```javascript
export type WriteFunction = (writeValue: string, entryOffset: number, encoding: string) => Promise<void>
```

#### XMP Removal

One of the metadata formats from which this package removes metadata is Adobe's [XMP](https://www.adobe.com/products/xmp.html).

Currently, if it finds XMP metadata in a file, this package simply wipes the whole of the XMP block rather than just the GPS metadata in that block; this code was originally written under a bit of a time crunch and removing all of the XMP was acceptable since we don't use XMP at all.

If you need to leave XMP intact, pass the optional `options` parameter object to `removeLocation` with `skipXMPRemoval` set to `true`:

```javascript
const gpsWasRemoved = await removeLocation(destPath, read, write, { skipXMPRemoval })
```

At some point I'll write (or accept a PR for) the logic to properly find and remove just the GPS from XMP and leave the rest.

## Testing

`nodeStripContent.js` is a node utility that takes a file name, source directory, and destination directory,
copies the file from the source to destination directory, and performs the metadata stripping operation
on the file in the destination directory, the result of which can then be compared to the original.

There is a command-line wrapper script around this called `batchRemoveMetadata.js` that takes an in-directory
and out-directory and performs this operation on all file in the in-directory, so a batch of files can manually
have GPS removed. This can be run with `yarn batch-remove-content`

There is also a jest suite around `nodeStripContent.js` providing the tests which all new commits must pass.
When this was an internal tool, this was designed to be used with an s3 bucket to which all XOi devs have access.
In open-sourcing this library, I've included a smaller set of test content instead since everything will instead
need to be checked in to git. The test suite runs content stripping against unprocessed files and compares them to
previously verified processed files and ensures consistent operation each time, as well as testing for corruption
and visual correctness for images. The tester can also add new content to the test collection. Manually run content
with the batch remover, manually check for visual correctness and metadata removal, then add the unprocessed file
to the corresponding `preprocessed` and the processed to `processed-clean` for its filetype.

Note that to run the jest tests you must `brew install graphicsmagick` on your machine.
