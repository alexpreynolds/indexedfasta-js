# @gmod/gff

[![Generated with nod](https://img.shields.io/badge/generator-nod-2196F3.svg?style=flat-square)](https://github.com/diegohaz/nod)
[![NPM version](https://img.shields.io/npm/v/@gmod/gff.svg?style=flat-square)](https://npmjs.org/package/@gmod/gff)
[![Build Status](https://img.shields.io/travis/GMOD/gff-js/master.svg?style=flat-square)](https://travis-ci.org/GMOD/gff-js) [![Coverage Status](https://img.shields.io/codecov/c/github/GMOD/gff-js/master.svg?style=flat-square)](https://codecov.io/gh/GMOD/gff-js/branch/master)

Read and write GFF3 data performantly. This module aims to be a complete implementation of the [GFF3 specification](https://github.com/The-Sequence-Ontology/Specifications/blob/master/gff3.md).

* streaming parsing and streaming formatting
* proper escaping and unescaping of attribute and column values
* supports features with multiple locations and features with multiple parents
* reconstructs feature hierarchies of both `Parent` and `Derives_from` relationships
* parses FASTA sections
* does no validation except for `Parent` and `Derives_from` relationships
* only compatible with GFF3

## Install

    $ npm install --save @gmod/gff

## Usage

```js
const gff = require('@gmod/gff').default
// or in ES6 (recommended)
import gff from '@gmod/gff'

// parse a file from a file name
// parses only features and sequences by default,
// set options to parse directives and/or comments
gff.parseFile('path/to/my/file.gff3', { parseAll: true })
.on('data', data => {
  if (data.directive) {
    console.log('got a directive',data)
  }
  else if (data.comment) {
    console.log('got a comment',data)
  }
  else if (data.sequence) {
    console.log('got a sequence from a FASTA section')
  }
  else {
    console.log('got a feature',data)
  }
})

// parse a stream of GFF3 text
const fs = require('fs')
fs.createReadStream('path/to/my/file.gff3')
.pipe(gff.parseStream())
.on('data', data => {
  console.log('got item',data)
  return data
})
.on('end', () => {
  console.log('done parsing!')
})

// parse a string of gff3 synchronously
let stringOfGFF3 = fs
  .readFileSync('my_annotations.gff3')
  .toString()
let arrayOfThings = gff.parseStringSync(stringOfGFF3)

// format an array of items to a string
let stringOfGFF3 = gff.formatSync(arrayOfThings)

// format a stream of things to a stream of text.
// inserts sync marks automatically.
myStreamOfGFF3Objects
.pipe(gff.formatStream())
.pipe(fs.createWriteStream('my_new.gff3'))

// format a stream of things and write it to
// a gff3 file. inserts sync marks and a
// '##gff-version 3' header if one is not
// already present
myStreamOfGFF3Objects
.pipe(gff.formatFile('path/to/destination.gff3')
```

## Object format

### features

In GFF3, features can have more than one location. We parse features
as arrayrefs of all the lines that share that feature's ID.
Values that are `.` in the GFF3 are `null` in the output.

A simple feature that's located in just one place:

```json
[
  {
    "seq_id": "ctg123",
    "source": null,
    "type": "gene",
    "start": 1000,
    "end": 9000,
    "score": null,
    "strand": "+",
    "phase": null,
    "attributes": {
      "ID": [
        "gene00001"
      ],
      "Name": [
        "EDEN"
      ]
    },
    "child_features": [],
    "derived_features": []
  }
```

A CDS called `cds00001` located in two places:

```json
[
  {
    "seq_id": "ctg123",
    "source": null,
    "type": "CDS",
    "start": 1201,
    "end": 1500,
    "score": null,
    "strand": "+",
    "phase": "0",
    "attributes": {
      "ID": [
        "cds00001"
      ],
      "Parent": [
        "mRNA00001"
      ]
    },
    "child_features": [],
    "derived_features": []
  },
  {
    "seq_id": "ctg123",
    "source": null,
    "type": "CDS",
    "start": 3000,
    "end": 3902,
    "score": null,
    "strand": "+",
    "phase": "0",
    "attributes": {
      "ID": [
        "cds00001"
      ],
      "Parent": [
        "mRNA00001"
      ]
    },
    "child_features": [],
    "derived_features": []
  }
]
```

### directives

```js
parseDirective("##gff-version 3\n")
// returns
{
  "directive": "gff-version",
  "value": "3"
}
```

```js
parseDirective('##sequence-region ctg123 1 1497228\n')
// returns
{
  "directive": "sequence-region",
  "value": "ctg123 1 1497228",
  "seq_id": "ctg123",
  "start": "1",
  "end": "1497228"
}
```

### comments

```js
parseComment('# hi this is a comment\n')
// returns
{
  "comment": "hi this is a comment"
}
```

### sequences

These come from any embedded `##FASTA` section in the GFF3 file.

```js
{
  "id": "ctgA",
  "description": "test contig",
  "sequence": "ACTGACTAGCTAGCATCAGCGTCGTAGCTATTATATTACGGTAGCCA"
}
```


## API

<!-- Generated by documentation.js. Update this documentation by updating the source code. -->

-   [parseStream](#parsestream)
-   [parseFile](#parsefile)
-   [parseStringSync](#parsestringsync)
-   [formatSync](#formatsync)
-   [formatStream](#formatstream)
-   [formatFile](#formatfile)

### parseStream

Parse a stream of text data into a stream of feature,
directive, and comment objects.

**Parameters**

-   `options` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)** optional options object (optional, default `{}`)
    -   `options.encoding` **[string](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** text encoding of the input GFF3. default 'utf8'
    -   `options.parseAll` **[boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** default false.  if true, will parse all items. overrides other flags
    -   `options.parseFeatures` **[boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** default true
    -   `options.parseDirectives` **[boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** default false
    -   `options.parseComments` **[boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** default false
    -   `options.parseSequences` **[boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** default true
    -   `options.bufferSize` **[Number](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Number)** maximum number of GFF3 lines to buffer. defaults to 1000

Returns **ReadableStream** stream (in objectMode) of parsed items

### parseFile

Read and parse a GFF3 file from the filesystem.

**Parameters**

-   `filename` **[string](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** the filename of the file to parse
-   `options` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)** optional options object
    -   `options.encoding` **[string](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** the file's string encoding, defaults to 'utf8'
    -   `options.parseAll` **[boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** default false.  if true, will parse all items. overrides other flags
    -   `options.parseFeatures` **[boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** default true
    -   `options.parseDirectives` **[boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** default false
    -   `options.parseComments` **[boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** default false
    -   `options.parseSequences` **[boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** default true
    -   `options.bufferSize` **[Number](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Number)** maximum number of GFF3 lines to buffer. defaults to 1000

Returns **ReadableStream** stream (in objectMode) of parsed items

### parseStringSync

Synchronously parse a string containing GFF3 and return
an arrayref of the parsed items.

**Parameters**

-   `str` **[string](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** 
-   `inputOptions` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)** optional options object (optional, default `{}`)
    -   `inputOptions.parseAll` **[boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** default false.  if true, will parse all items. overrides other flags
    -   `inputOptions.parseFeatures` **[boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** default true
    -   `inputOptions.parseDirectives` **[boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** default false
    -   `inputOptions.parseComments` **[boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** default false
    -   `inputOptions.parseSequences` **[boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** default true

Returns **[Array](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Array)** array of parsed features, directives, and/or comments

### formatSync

Format an array of GFF3 items (features,directives,comments) into string of GFF3.
Does not insert synchronization (###) marks.

**Parameters**

-   `items`  

Returns **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** the formatted GFF3

### formatStream

Format a stream of items (of the type produced
by this script) into a stream of GFF3 text.

Inserts synchronization (###) marks automatically.

**Parameters**

-   `options` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)** 
    -   `options.minSyncLines` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)** minimum number of lines between ### marks. default 100
    -   `options.insertVersionDirective` **[Boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** if the first item in the stream is not a ##gff-version directive, insert one.
         default false

### formatFile

Format a stream of items (of the type produced
by this script) into a GFF3 file and write it to the filesystem.

Inserts synchronization (###) marks and a ##gff-version
directive automatically (if one is not already present).

**Parameters**

-   `stream` **ReadableStream** the stream to write to the file
-   `filename` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** the file path to write to
-   `options` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)**  (optional, default `{}`)
    -   `options.encoding` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** default 'utf8'. encoding for the written file
    -   `options.minSyncLines` **[Number](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Number)** minimum number of lines between sync (###) marks. default 100
    -   `options.insertVersionDirective` **[Boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** if the first item in the stream is not a ##gff-version directive, insert one.
         default true

Returns **[Promise](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise)** promise for the written filename

## util

There is also a `util` module that contains super-low-level functions for dealing with lines and parts of lines.

```js
const util = require('@gmod/gff/util')
const gff3Lines = util.formatItem({
  seq_id: 'ctgA',
  ...
}))
```

-   [unescape](#unescape)
-   [escape](#escape)
-   [parseAttributes](#parseattributes)
-   [parseFeature](#parsefeature)
-   [parseDirective](#parsedirective)
-   [formatAttributes](#formatattributes)
-   [formatFeature](#formatfeature)
-   [formatDirective](#formatdirective)
-   [formatComment](#formatcomment)
-   [formatSequence](#formatsequence)
-   [formatItem](#formatitem)

### unescape

Unescape a string value used in a GFF3 attribute.

**Parameters**

-   `s` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** 

Returns **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** 

### escape

Escape a value for use in a GFF3 attribute value.

**Parameters**

-   `s` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** 

Returns **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** 

### parseAttributes

Parse the 9th column (attributes) of a GFF3 feature line.

**Parameters**

-   `attrString` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** 

Returns **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)** 

### parseFeature

Parse a GFF3 feature line

**Parameters**

-   `line` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** 

### parseDirective

Parse a GFF3 directive line.

**Parameters**

-   `line` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** 

Returns **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)** the information in the directive

### formatAttributes

Format an attributes object into a string suitable for the 9th column of GFF3.

**Parameters**

-   `attrs` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)** 

### formatFeature

Format a feature object or array of
feature objects into one or more lines of GFF3.

**Parameters**

-   `featureOrFeatures`  

### formatDirective

Format a directive into a line of GFF3.

**Parameters**

-   `directive` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)** 

Returns **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** 

### formatComment

Format a comment into a GFF3 comment.
Yes I know this is just adding a # and a newline.

**Parameters**

-   `comment` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)** 

Returns **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** 

### formatSequence

Format a sequence object as FASTA

**Parameters**

-   `seq` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)** 

Returns **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** formatted single FASTA sequence

### formatItem

Format a directive, comment, or feature,
or array of such items, into one or more lines of GFF3.

**Parameters**

-   `itemOrItems` **([Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object) \| [Array](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Array))** 

## License

MIT © [Robert Buels](https://github.com/rbuels)
