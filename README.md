# stream-json

[![Build status][travis-image]][travis-url]
[![Dependencies][deps-image]][deps-url]
[![devDependencies][dev-deps-image]][dev-deps-url]
[![NPM version][npm-image]][npm-url]
[![TypeScript definitions on DefinitelyTyped][definitelytyped-image]](definitelytyped-url)


`stream-json` is a micro-library of node.js stream components with minimal dependencies for creating custom data processors oriented on processing huge JSON files while requiring a minimal memory footprint. It can parse JSON files far exceeding available memory. Even individual primitive data items (keys, strings, and numbers) can be streamed piece-wise. Streaming SAX-inspired event-based API is included as well.

Available components:

* Streaming JSON [Parser](https://github.com/uhop/stream-json/wiki/Parser).
  * It produces a SAX-like token stream.
  * Optionally it can pack keys, strings, and numbers (controlled separately).
  * The [main module](https://github.com/uhop/stream-json/wiki/Main-module) provides helpers to create a parser.
* Filters to edit a token stream:
  * [Pick](https://github.com/uhop/stream-json/wiki/Pick) selects desired objects.
  * [Replace](https://github.com/uhop/stream-json/wiki/Replace) substitutes objects with a replacement.
  * [Ignore](https://github.com/uhop/stream-json/wiki/Ignore) removes objects.
  * [Filter](https://github.com/uhop/stream-json/wiki/Filter) filters tokens maintaining stream's validity.
* Streamers to produce a stream of JavaScript objects.
  * [StreamArray](https://github.com/uhop/stream-json/wiki/StreamArray) takes an array of objects and produces a stream of its components.
    * It streams array components individually taking care of assembling them automatically.
    * Created initially to deal with JSON files similar to [Django](https://www.djangoproject.com/)-produced database dumps.
  * [StreamObject](https://github.com/uhop/stream-json/wiki/StreamObject) takes an object and produces a stream of its top-level properties.
  * [StreamValues](https://github.com/uhop/stream-json/wiki/StreamValues) can handle a stream of JSON object.
    * It supports [JSON Streaming](https://en.wikipedia.org/wiki/JSON_Streaming) protocol, where individual values are separated semantically (like in `"{}[]"`), or with whitespaces (like in `"true 1 null"`).
    * Useful to stream objects selected by `Pick`, or generated by other means.
* Essentials:
  * [Assembler](https://github.com/uhop/stream-json/wiki/Assembler) interprets a token stream creating JavaScript objects.
  * [Disassembler](https://github.com/uhop/stream-json/wiki/Disassembler) produces a token stream from JavaScript objects.
  * [Stringer](https://github.com/uhop/stream-json/wiki/Stringer) converts a token stream back into a JSON text stream.
  * [Emitter](https://github.com/uhop/stream-json/wiki/Emitter) reads a token stream and emits each token as an event.
    * It can greatly simplify data processing.
* Utilities:
  * [emit()](https://github.com/uhop/stream-json/wiki/emit()) makes any stream component to emit tokens as events.
  * [withParser()](https://github.com/uhop/stream-json/wiki/withParser()) helps to create stream components with a parser.

All components are meant to be building blocks to create flexible custom data processing pipelines. They can be extended and/or combined with custom code. They can be used together with [stream-chain](https://www.npmjs.com/package/stream-chain) to simplify data processing.

This toolkit is distributed under New BSD license.

## Introduction

```js
const {chain}  = require('stream-chain');

const {parser} = require('stream-json');
const {pick}   = require('stream-json/filters/Pick');
const {ignore} = require('stream-json/filters/Ignore');
const {StreamValues} = require('stream-json/streamers/StreamValues');

const fs   = require('fs');
const zlib = require('zlib');

const pipeline = chain([
  fs.createReadStream('sample.json.gz'),
  zlib.createGunzip(),
  parser(),
  pick({filter: 'data'}),
  ignore({filter: /\b_meta\b/i}),
  streamValues(),
  data => {
    const value = data.value;
    return value && value.department === 'accounting' ? data : null;
  }
]);

let counter = 0;
pipeline.on('data', () => ++counter);
pipeline.on('end', () =>
  console.log(`The accounting department has ${counter} employees.`));
```

See the full documentation in [Wiki](https://github.com/uhop/stream-json/wiki).

Companion projects:

* [stream-csv-as-json](https://www.npmjs.com/package/stream-csv-as-json) streams huge CSV files in a format compatble with `stream-json`: rows as arrays of string values. If a header row is used, it can stream rows as objects with named fields.

## Installation

```bash
npm install --save stream-json
# or:
yarn add stream-json
```

## Use

The whole library is organized as a set of small components, which can be combined to produce the most effective pipeline. All components are based on node.js [streams](http://nodejs.org/api/stream.html), and [events](http://nodejs.org/api/events.html). They implement all required standard APIs. It is easy to add your own components to solve your unique tasks.

The code of all components is compact and simple. Please take a look at their source code to see how things are implemented, so you can produce your own components in no time.

Obviously, if a bug is found, or a way to simplify existing components, or new generic components are created, which can be reused in a variety of projects, don't hesitate to open a ticket, and/or create a pull request.

## Release History

- 1.1.0 *added `Disassembler`.*
- 1.0.3 *minor tweaks, added TypeScript typings and the badge.*
- 1.0.2 *minor tweaks, documentation improvements.*
- 1.0.1 *reorg to fix export problems.*
- 1.0.0 *the first 1.0 release.*
- 0.6.1 *the technical release.*
- 0.6.0 *added Stringer to convert event streams back to JSON.*
- 0.5.3 *bug fix to allow empty JSON Streaming.*
- 0.5.2 *bug fixes in `Filter`.*
- 0.5.1 *corrected README.*
- 0.5.0 *added support for [JSON Streaming](https://en.wikipedia.org/wiki/JSON_Streaming).*
- 0.4.2 *refreshed dependencies.*
- 0.4.1 *added `StreamObject` by [Sam Noedel](https://github.com/delta62).*
- 0.4.0 *new high-performant Combo component, switched to the previous parser.*
- 0.3.0 *new even faster parser, bug fixes.*
- 0.2.2 *refreshed dependencies.*
- 0.2.1 *added utilities to filter objects on the fly.*
- 0.2.0 *new faster parser, formal unit tests, added utilities to assemble objects on the fly.*
- 0.1.0 *bug fixes, more documentation.*
- 0.0.5 *bug fixes.*
- 0.0.4 *improved grammar.*
- 0.0.3 *the technical release.*
- 0.0.2 *bug fixes.*
- 0.0.1 *the initial release.*

[npm-image]:      https://img.shields.io/npm/v/stream-json.svg
[npm-url]:        https://npmjs.org/package/stream-json
[deps-image]:     https://img.shields.io/david/uhop/stream-json.svg
[deps-url]:       https://david-dm.org/uhop/stream-json
[dev-deps-image]: https://img.shields.io/david/dev/uhop/stream-json.svg
[dev-deps-url]:   https://david-dm.org/uhop/stream-json?type=dev
[travis-image]:   https://img.shields.io/travis/uhop/stream-json.svg
[travis-url]:     https://travis-ci.org/uhop/stream-json
[definitelytyped-image]: https://img.shields.io/badge/DefinitelyTyped-.d.ts-blue.svg
[definitelytyped-url]:   https://definitelytyped.org
