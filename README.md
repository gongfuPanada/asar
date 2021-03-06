# asar - Atom-Shell Archive

[![build status](http://img.shields.io/travis/atom/asar.svg?style=flat-square)](https://travis-ci.org/atom/asar)
[![dependencies](http://img.shields.io/david/atom/asar.svg?style=flat-square)](https://david-dm.org/atom/asar)
[![npm version](http://img.shields.io/npm/v/asar.svg?style=flat-square)](https://npmjs.org/package/asar)

Asar is a simple extensive archive format, it works like `tar` that concatenates
all files together without compression, while having random access support.

## Features

* Support random access
* Use JSON to store files' information
* Very easy to write a parser

## Command line utility

### Install

```bash
$ npm install asar
```

### Usage

```bash
$ asar --help

  Usage: asar [options] [command]

  Commands:

    pack|p <dir> <output>
       create asar archive

    list|l <archive>
       list files of asar archive

    extract-file|ef <archive> <filename>
       extract one file from archive

    extract|e <archive> <dest>
       extract archive


  Options:

    -h, --help     output usage information
    -V, --version  output the version number

```

## Using programatically

### Example

```js
var asar = require('asar');

var src = 'some/path/';
var dest = 'name.asar';

asar.createPackage(src, dest, function() {
  console.log('done.');
})
```

Please note that there is currently *no* error handling provided!

## Using with grunt

There is also an inofficial grunt plugin to generate asar archives at [bwin/grunt-asar][grunt-asar].

## Format

Asar uses [Pickle][pickle] to safely serialize binary value to file, there is
also a [node.js binding][node-pickle] of `Pickle` class.

The format of asar is very flat:

```
| UInt32: header_size | String: header | Bytes: file1 | ... | Bytes: file42 |
```

The `header_size` and `header` are serialized with [Pickle][pickle] class, and
`header_size`'s [Pickle][pickle] object is 8 bytes.

The `header` is a JSON string, and the `header_size` is the size of `header`'s
`Pickle` object.

Structure of `header` is something like this:

```json
{
   "files": {
      "tmp": {
         "files": {}
      },
      "usr" : {
         "files": {
           "bin": {
             "files": {
               "ls": {
                 "offset": "0",
                 "size": 100,
                 "executable": true
               },
               "cd": {
                 "offset": "100",
                 "size": 100,
                 "executable": true
               }
             }
           }
         }
      },
      "etc": {
         "files": {
           "hosts": {
             "offset": "200",
             "size": 32
           }
         }
      }
   }
}
```

`offset` and `size` records the information to read the file from archive, the
`offset` starts from 0 so you have to manually add the size of `header_size` and
`header` to the `offset` to get the real offset of the file.

`offset` is a UINT64 number represented in string, because there is no way to
precisely represent UINT64 in JavaScript `Number`. `size` is a JavaScript
`Number` that is no larger than `Number.MAX_SAFE_INTEGER`, which has a value of
`9007199254740991` and is about 8PB in size. We didn't store `size` in UINT64
because file size in Node.js is represented as `Number` and it is not safe to
convert `Number` to UINT64.

[pickle]: https://chromium.googlesource.com/chromium/src/+/master/base/pickle.h
[node-pickle]: https://www.npmjs.org/package/chromium-pickle
[grunt-asar]: https://github.com/bwin/grunt-asar
