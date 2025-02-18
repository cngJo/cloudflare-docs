---
pcx_content_type: configuration
title: Streams
---

# Streams

{{<render file="_nodejs-compat-howto.md">}}

The [Node.js streams API](https://nodejs.org/api/stream.html) is the original API for working with streaming data in JavaScript, predating the [WHATWG ReadableStream standard](https://streams.spec.whatwg.org/). A stream is an abstract interface for working with streaming data in Node.js. Streams can be readable, writable, or both. All streams are instances of [EventEmitter](/workers/runtime-apis/nodejs/eventemitter/).

Where possible, you should use the [WHATWG standard "Web Streams" API](https://streams.spec.whatwg.org/), which is [supported in Workers](https://streams.spec.whatwg.org/).

```js
import {
  Readable,
  Transform,
} from 'node:stream';

import {
  text,
} from 'node:stream/consumers';

import {
  pipeline,
} from 'node:stream/promises';

// A Node.js-style Transform that converts data to uppercase
// and appends a newline to the end of the output.
class MyTransform extends Transform {
  constructor() {
    super({ encoding: 'utf8' });
  }
  _transform(chunk, _, cb) {
    this.push(chunk.toString().toUpperCase());
    cb();
  }
  _flush(cb) {
    this.push('\n');
    cb();
  }
}

export default {
  async fetch() {
    const chunks = [
      "hello ",
      "from ",
      "the ",
      "wonderful ",
      "world ",
      "of ",
      "node.js ",
      "streams!"
    ];

    function nextChunk(readable) {
      readable.push(chunks.shift());
      if (chunks.length === 0) readable.push(null);
      else queueMicrotask(() => nextChunk(readable));
    }

    // A Node.js-style Readable that emits chunks from the
    // array...
    const readable = new Readable({
      encoding: 'utf8',
      read() { nextChunk(readable); }
    });

    const transform = new MyTransform();
    await pipeline(readable, transform);
    return new Response(await text(transform));
  }
};
```

Refer to the [Node.js documentation for `stream`](https://nodejs.org/api/stream.html) for more information.