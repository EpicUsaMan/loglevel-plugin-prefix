# loglevel-prefix
Minimal, dependency-free and lightweight (0.9KB minified and gzipped) plugin for [loglevel](https://github.com/pimterry/loglevel) message prefixing

## Installation

```sh
npm install loglevel-prefix --save
```

## API

```javascript
prefix(loglevel[, options]);
```

**loglevel** - root logger object, imported from loglevel package

**options** - configuration object

## Base usage

### Browser directly

Download [production version](https://raw.githubusercontent.com/kutuluk/loglevel-prefix/master/dist/loglevel-prefix.min.js)
and copy to your project folder
```html
<script src="loglevel.min.js"></script>
<script src="loglevel-prefix.min.js"></script>

<script>
  prefix(log);
  log.warn('prefixed message');
</script>
```

### ES6
```javascript

import log from 'loglevel';
import prefix from 'loglevel-prefix';

prefix(log);
log.warn('prefixed message');

```

### CommonJS
```javascript

var log = require('loglevel');
var prefix = require('loglevel-prefix');
prefix(log);

// or
// var log = require('loglevel-prefix')(require('loglevel'));

log.warn('prefixed message');

```

### AMD
```javascript
define(['loglevel', 'loglevel-prefix'], function(log, prefix) {
  prefix(log);
  log.warn('prefixed message');
});
```

## Options

### Default options
```javascript
default_options = {
  template: '[%t] %l:',
  dateFormatter: date => date.toTimeString().replace(/.*(\d{2}:\d{2}:\d{2}).*/, '$1'),
  levelFormatter: level => level.toUpperCase(),
  nameFormatter: name => name || 'root'
}
```

Plugin formats the prefix using template option as a printf-like format.

The **template** is a string containing zero or more placeholder tokens. Each placeholder token is replaced with the value from loglevel messages parameters. Supported placeholders are:

%t - Timestamp of message.

%l - Level of message.

%n - Name of logger.

The **dateFormatter, levelFormatter, nameFormatter** is a functions for formatting corresponding values

```javascript

import log from 'loglevel';
import prefix from 'loglevel-prefix';

prefix(log);
log.warn('prefixed message');

```
Output
```
[12:53:46] WARN: prefixed message
```

### Custom options
```javascript
import log from 'loglevel';
import prefix from 'loglevel-prefix';

prefix(log, {
  template: '[%t] %l (%n) static text:',
  dateFormatter: date => date.toISOString(),
  levelFormatter: level => level.charAt(0).toUpperCase() + level.substr(1),
  nameFormatter: name => name || 'global'
});

log.warn('prefixed message');
```

Output
```
[2017-05-29T16:53:46.000Z] Warn (global) static text: prefixed message
```

## Usage

```javascript
// moduleA.js
import log from 'loglevel';

export default function () {
  log.warn('message from moduleA');
}
```

```javascript
// moduleB.js
import loglevel from 'loglevel';
const log = loglevel.getLogger('moduleB');

export default function () {
  log.warn('message from moduleB');
}
```

```javascript
// moduleC.js
import loglevel from 'loglevel';

export default function () {
  const log = loglevel.getLogger('moduleC');
  log.warn('message from moduleC');
}
```

```javascript
// main.js
import log from 'loglevel';
import prefix from 'loglevel-prefix';

import a from './moduleA';
import b from './moduleB';
import c from './moduleC';

log.warn('message from root %s prefixing', 'before');

prefix(log, {
  template: '[%t] %l (%n):',
});

log.warn('message from root %s prefixing', 'after');

a();
b();
c();
```

Output
```
message from root before prefixing
[16:53:46] WARN (root): message from root after prefixing
[16:53:46] WARN (root): message from moduleA
message from moduleB
[16:53:46] WARN (moduleC): message from moduleC
```

## Errors

```javascript
// main.js
import loglevel from 'loglevel';
import prefix from 'loglevel-prefix';

prefix(loglevel);

loglevel.warn('message from root after prefixing');

prefix(loglevel, {
  dateFormatter: date => date.toISOString()
});

loglevel.warn('message from root after pre-prefixing');

const log = loglevel.getLogger('main');

prefix(log, {
  template: '[%t] %l (%n):'
});

log.warn('message from child logger');
```

Output
```
[16:53:46] WARN: message from root after prefixing
Uncaught TypeError: You can assign a prefix only one time
[16:53:46] WARN: message from root after pre-prefixing
Uncaught TypeError: Argument is not a root loglevel object
[16:53:46] WARN: message from child logger
```
