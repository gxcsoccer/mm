mm
=======

[![NPM version][npm-image]][npm-url]
[![build status][travis-image]][travis-url]
[![Test coverage][coveralls-image]][coveralls-url]
[![Gittip][gittip-image]][gittip-url]
[![David deps][david-image]][david-url]
[![node version][node-image]][node-url]
[![npm download][download-image]][download-url]

[npm-image]: https://img.shields.io/npm/v/mm.svg?style=flat-square
[npm-url]: https://npmjs.org/package/mm
[travis-image]: https://img.shields.io/travis/node-modules/mm.svg?style=flat-square
[travis-url]: https://travis-ci.org/node-modules/mm
[coveralls-image]: https://img.shields.io/coveralls/node-modules/mm.svg?style=flat-square
[coveralls-url]: https://coveralls.io/r/node-modules/mm?branch=master
[gittip-image]: https://img.shields.io/gittip/fengmk2.svg?style=flat-square
[gittip-url]: https://www.gittip.com/fengmk2/
[david-image]: https://img.shields.io/david/node-modules/mm.svg?style=flat-square
[david-url]: https://david-dm.org/node-modules/mm
[node-image]: https://img.shields.io/badge/node.js-%3E=_0.10-green.svg?style=flat-square
[node-url]: http://nodejs.org/download/
[download-image]: https://img.shields.io/npm/dm/mm.svg?style=flat-square
[download-url]: https://npmjs.org/package/mm

An simple but flexible **mock(or say stub)** package, mock mate.

![logo](https://raw.github.com/node-modules/mm/master/logo.png)

## Install

```bash
$ npm install mm
```

## Usage

```js
var mm = require('mm');
var fs = require('fs');

mm(fs, 'readFileSync', function (filename) {
  return filename + ' content';
});

console.log(fs.readFileSync('《九评 Java》'));
// => 《九评 Java》 content

mm.restore();

console.log(fs.readFileSync('《九评 Java》'));
// => throw `Error: ENOENT, no such file or directory '《九评 Java》`
```

### Support generator function

```js
var foo = {
  get: function* () {
    return 1;
  }
};

mm.data(foo, 'get', 2);
var data = yield* foo.get(); // data should return 2

mm.error(foo, 'get', 'error boom');
yield* foo.get(); // should throw error
```

## API

### .error(module, propertyName, errerMessage)

```js
var mm = require('mm');
var fs = require('fs');

mm.error(fs, 'readFile', 'mock fs.readFile return error');

fs.readFile('/etc/hosts', 'utf8', function (err, content) {
  // err.name === 'MockError'
  // err.message === 'mock fs.readFile return error'
  console.log(err);

  mm.restore(); // remove all mock effects.

  fs.readFile('/etc/hosts', 'utf8', function (err, content) {
    console.log(err); // => null
    console.log(content); // => your hosts
  });
});
```

### .data(module, propertyName, secondCallbackArg)

```js
mm.data(fs, 'readFile', new Buffer('some content'));

// equals

fs.readFile = function (args..., callback) {
  callback(null, new Buffer('some content'))
};
```

### .empty(module, propertyName)

```js
mm.empty(mysql, 'query');

// equals

mysql.query = function (args..., callback) {
  callback();
}
```

### .datas(module, propertyName, argsArray)

```js
mm.datas(urllib, 'request', [new Buffer('data'), {headers: { foo: 'bar' }}]);

// equals

urllib.request = function (args..., callback) {
  callback(null, new Buffer('data'), {headers: { foo: 'bar' }});
}
```

### .restore()

```js
// restore all mock properties
mm.restore();
```

### .http.request(mockUrl, mockResData, mockResHeaders) and .https.request(mockUrl, mockResData, mockResHeaders)

```js
var mm = require('mm');
var http = require('http');

var mockURL = '/foo';
var mockResData = 'mock data';
var mockResHeaders = { server: 'mock server' };
mm.http.request(mockURL, mockResData, mockResHeaders);
mm.https.request(mockURL, mockResData, mockResHeaders);

// http
http.get({
  path: '/foo'
}, function (res) {
  console.log(res.headers); // should be mock headers
  var body = '';
  res.on('data', function (chunk) {
    body += chunk.toString();
  });
  res.on('end', function () {
    console.log(body); // should equal 'mock data'
  });
});

// https
https.get({
  path: '/foo'
}, function (res) {
  console.log(res.headers); // should be mock headers
  var body = '';
  res.on('data', function (chunk) {
    body += chunk.toString();
  });
  res.on('end', function () {
    console.log(body); // should equal 'mock data'
  });
});
```

### .http.requestError(mockUrl, reqError, resError)

```js
var mm = require('mm');
var http = require('http');

var mockURL = '/foo';
var reqError = null;
var resError = 'mock res error';
mm.http.requestError(mockURL, reqError, resError);

var req = http.get({
  path: '/foo'
}, function (res) {
  console.log(res.statusCode, res.headers); // 200 but never emit `end` event
  res.on('end', fucntion () {
    console.log('never show this message');
  });
});
req.on('error', function (err) {
  console.log(err); // should return mock err: err.name === 'MockHttpResponseError'
}
```

## Authors

* fengmk2 <https://github.com/fengmk2>
* dead-horse <https://github.com/dead-horse>
* alsotang <https://github.com/alsotang>

## License

This software is licensed under the MIT License.

Copyright (C) 2012 - 2014 fengmk2 <fengmk2@gmail.com>

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
