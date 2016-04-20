flake-idgen-63
===========
[![npm version](https://img.shields.io/npm/v/flake-idgen-63.svg)](https://www.npmjs.com/package/flake-idgen-63)
[![Build Status](https://travis-ci.org/luxe-eng/flake-idgen-63.svg?branch=master)](https://travis-ci.org/luxe-eng/flake-idgen-63)
[![Coverage Status](https://coveralls.io/repos/github/luxe-eng/flake-idgen-63/badge.svg?branch=master)](https://coveralls.io/github/luxe-eng/flake-idgen-63?branch=master)
[![Code Climate](https://codeclimate.com/github/luxe-eng/flake-idgen-63/badges/gpa.svg)](https://codeclimate.com/github/luxe-eng/flake-idgen-63)
[![Code Documentation](http://inch-ci.org/github/luxe-eng/flake-idgen-63.svg?branch=master&style=shields)](http://inch-ci.org/github/luxe-eng/flake-idgen-63)
[![Issue Count](https://codeclimate.com/github/luxe-eng/flake-idgen-63/badges/issue_count.svg)](https://codeclimate.com/github/luxe-eng/flake-idgen-63)
[![Dependency Status](https://david-dm.org/luxe-eng/flake-idgen-63.svg)](https://david-dm.org/luxe-eng/flake-idgen-63)
[![npm downloads](https://img.shields.io/npm/dm/flake-idgen-63.svg)](https://www.npmjs.com/package/flake-idgen-63)
[![GitHub Issues](https://img.shields.io/github/issues/luxe-eng/flake-idgen-63.svg)](https://github.com/luxe-eng/flake-idgen-63/issues?q=is%3Aopen)
[![License](https://img.shields.io/npm/l/flake-idgen-63.svg)](LICENSE.txt)
[![Cat Gifs](https://img.shields.io/badge/powered%20by-cat%20gifs%20%F0%9F%90%88-brightgreen.svg)](http://giphy.com/search/cat)

flake-idgen-63 yields 64-Bit (with 63 signigicant bits) k-ordered, conflict-free ids in a distributed environment.

## Installation

```
$ npm install --save flake-idgen-63 ⏎
```

## Format ##

The flake-idgen-63 format is made up of: `timestamp`, `datacenter`, `worker` and `counter`. Examples in the following table: 
```
+--------------------------+------------+--------+---------+---------------------+
|        Timestamp         | Datacenter | Worker | Counter |   Flake ID (HEX)    |
+--------------------------+------------+--------+---------+---------------------+
| [2013, 3, 1, 0, 0, 0, 0] |      0     |    0   |    0    | 27b8 5c96 0000 0000 |
+--------------------------+------------+--------+---------+---------------------+
| [2013, 3, 1, 0, 0, 0, 0] |      0     |    0   |    1    | 27b8 5c96 0000 0001 |
+--------------------------+------------+--------+---------+---------------------+
| [2013, 3, 1, 0, 0, 0, 0] |      0     |    0   |    2    | 27b8 5c96 0000 0002 |
+--------------------------+------------+--------+---------+---------------------+
| [2013, 3, 1, 0, 0, 0, 0] |      0     |    1   |    0    | 27b8 5c96 0000 1000 |
+--------------------------+------------+--------+---------+---------------------+
| [2013, 3, 1, 0, 0, 0, 0] |      0     |    1   |    1    | 27b8 5c96 0000 1001 |
+--------------------------+------------+--------+---------+---------------------+
| [2013, 3, 1, 0, 0, 0, 1] |      0     |    0   |    0    | 27b8 5c96 0020 0000 |
+--------------------------+------------+--------+---------+---------------------+
| [2013, 3, 1, 0, 0, 0, 1] |      0     |    0   |    1    | 27b8 5c96 0020 0001 |
+--------------------------+------------+--------+---------+---------------------+
| [2013, 3, 1, 0, 0, 0, 1] |      0     |    0   |    2    | 27b8 5c96 0020 0002 |
+--------------------------+------------+--------+---------+---------------------+
| [2013, 3, 1, 0, 0, 0, 1] |     15     |    1   |    0    | 27b8 5c96 003e 1000 |
+--------------------------+------------+--------+---------+---------------------+
| [2013, 3, 1, 0, 0, 0, 1] |     15     |    1   |    1    | 27b8 5c96 003e 1001 |
+--------------------------+------------+--------+---------+---------------------+
| [2013, 3, 1, 0, 0, 0, 1] |     15     |    1   |    2    | 27b8 5c96 003e 1002 |
+--------------------------+------------+--------+---------+---------------------+
```

As you can see, each Flake ID is 64 bits long, consisting of:
* `placeholder`, a 1 bit placeholder to avoid issues in languages without unsigned integer data types.
* `timestamp`, a 42 bit long number of milliseconds elapsed since 1 January 1970 00:00:00 UTC.
* `datacenter`, a 4 bit long datacenter identifier. It can take up to 16 unique values (including 0).
* `worker`, a 5 bit long worker indentifier. It can take up to 32 unique values (including 0).
* `counter`, a 12 bit long counter of ids in the same millisecond. It can take up to 4096 unique values. 

Breakdown of bits for an id e.g. `27b8 5c96 003e 1001` (datacenter is `15` and worker `1`, counter is `2`) is as follows:
```
 0 010 0111 1011 1000 0101 1100 1001 0110 0000 0000 001 1 111 0 0001 0000 0000 0010
                                                                    |---- ---- ----|  12 bit counter
                                                             |- ----|                  5 bit worker
                                                       |- ---|                         4 bit datacenter
  |--- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---|                              42 bit timestamp
|-|                                                                                    1 bit reserved
```

Note that composition of `datacenter id` and `worker id` makes 512 unique generator identifiers. By modifying datacenter and worker id we can get up to 512 id generators on a single machine (e.g. each running in a separate process) or have 512 machines with a single id generator on each. It is also possible to provide a single 9 bit long identifier (up to 512 values). That id is internally split into `datacenter` (the most significant 4 bits) and `worker` (the least significant 5 bits).

## Usage ##

Flake ID Generator returns 8 byte long node [Buffer](http://nodejs.org/api/buffer.html) objects with its bytes representing 64 bit long id. Note that the number is stored in Big Endian format i.e. the most significant byte of the number is stored in the smallest address given and the least significant byte is stored in the largest.

```js
var FlakeId63 = require('flake-idgen-63');
var flakeIdGen63 = new FlakeId63();

console.log(flakeIdGen63.next());
console.log(flakeIdGen63.next());
console.log(flakeIdGen63.next());
```

It would give something like:
```
<Buffer 27 b8 5c 96 00 20 00 00>
<Buffer 27 b8 5c 96 00 20 00 01>
<Buffer 27 b8 5c 96 00 20 00 02>
```

### Formatting ###

```js
var biguintformat = require('biguint-format');
const FlakeId63 = require('flake-idgen-63')

var flakeIdGen63 = new FlakeId63();

console.info(intformat(flakeIdGen63.next(), 'dec'));
console.info(intformat(flakeIdGen63.next(), 'hex', { groupsize: 2 }));
console.info(intformat(flakeIdGen63.next(), 'bin', { groupsize: 4 }));
```

It would give something like:
```js
2862139362510897152 // decimal
27 b8 5c 96 00 20 00 01  // hex
0010 0111 1011 1000 0101 1100 1001 0110 0000 0000 0010 0000 0000 0000 0000 0010  // binary
```