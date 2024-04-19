# Changelog

## v4.3.4 (2023-10-26)
------------------------

Fixed DatasetFactory adoption.

### 🔧 Fixes
 - `@graphy/core.data.factory` - Fixed a bug that was not correctly patching an adopted DatasetFactory with XSD-specific literal methods.


## v4.3.3 (2020-09-08)
------------------------

Fixed handling of prefixes in XML Scriber.

### 🔧 Fixes
 - `@graphy/content.xml.sribe` - Patched bug in XML Scriber when serializing prefixes that could lead to an infinite loop. Also, now prevents user from serializing any namespace beginning with 'xml' as these are reserved namespaces.
 - Upgraded dependency on `uri-js`.


## v4.3.2 (2020-08-28)
------------------------

Dependency upgrades and RDFJS compatibility fixes.

### 🔧 Fixes
 - Upgraded dependencies (and devDependencies), removed unused dependency on `uuid`.
 - Fixed RDFJS `.import` method returning the wrong stream.


## v4.3.1 (2020-08-27)
------------------------

Fixed XML Scriber normalizing quads from other libs.


### 🔧 Fixes
 - `@graphy/content.xml.scribe` - XML Scriber now normalizes quad-like objects before serializing.


## v4.3.0 (2020-08-22)
------------------------

Added [`.directives` option](https://graphy.link/content.textual#config_write) to writers.


### 🔧 Features
 - Added `.directives` option to writers.


## v4.2.1 (2020-05-05)
------------------------

Removed an unused dependency that could interfere with the installation on some platforms.


### 🔧 Fixes

#### Dependencies
 - Removed `worker`.


## v4.2.0 (2020-04-20)
------------------------

This update adds **experimental** multithreaded reading for N-Triples and N-Quads, called 'Scanners': `@graphy/content.nt.scan` and `@graphy/content.nq.scan`. These are deserializers that take a single input stream and read it across multiple workers in parallel, allowing users to take advantage of multiprocessor systems.

> `@graphy/content.nt.scan` and `@graphy/content.nq.scan` are experimental; they have limited test case coverage in node.js and have not been tested in the browser.

**How it works:**
The main thread populates sections of a preallocated `SharedArrayBuffer` with chunks copied from the input stream data and allows workers to claim each slot on a first-come-first-serve basis (synchronized using `Atomics`). For each chunk of input data, the main thread handles reading the leading and trailing fragments that belong to RDF statements spanning across chunk boundaries. Communication about data exchange is done mostly through unidirectional byte indicators on `SharedArrayBuffer`, with occassional help and synchronization from `Atomics` when necessary. Error handling and gathering of results is done using message passing, i.e., `postMessage`, which accepts transferable objects when passing large amounts of data.

**How its used:**
Similar to map/reduce, you must provide some code that will be run on each thread to create the reader instance and send updates or submit the final result(s) back to the main thread. Set the `.run` property on the config object passed to the constructor, it will be `eval`'d on both the main thread and each of the workers (`eval` is necessary to allow access to global context, such as `require`, while providing consistent behavior across node.js and browser). Tasks are therefore intended only for running trusted code. See [scan verb documentation](https://graphy.link/content.textual#verb_scribe) for more information.


### 🍭 Features

#### `@graphy/content.nt.scan` and `@graphy/content.nq.scan`
 - Introduced the [scan verb](https://graphy.link/content.textual#verb_scribe) for N-Triples and N-Quads.

#### CLI
 - Added the [`scan` command](https://graphy.link/cli#command_scan), which currently only supports internal piping to limited versions of the `count` and `scribe` commands.
 - Added [`--show-stack-trace` option](https://graphy.link/cli#options).
 - Added [`example`, `help` and `version` command aliases](https://graphy.link/cli#command_examples).


### ⚡︎ Performance

#### `@graphy/content.*.read`, `@graphy/content.*.write` and `@graphy/content.*.scribe`
 - Microtask optimizations.


### 🔧 Fixes

#### Dependencies
 - Dependency upgrades that affect CLI.


## v4.1.0 (2020-03-19)
------------------------

Added RDF/XML Serializer as Scriber: `@graphy/content.xml.scribe`.

### 🍭 Features

#### `@graphy/content.xml.scribe`
 - Introduced RDF/XML Content Scriber.

### 🔧 Fixes

#### `@graphy/content.t*.scribe` and `@graphy/content.t*.write`
 - Pre-emptively patched a situation that might have lead to serializing `undefined` during repeated `'prefix'` events.


## v4.0.5 (2020-02-11)
------------------------

Fixed missing dependency to `@graphy/core.class.scribable` for all writer packages.

### 🔧 Fixes

#### `@graphy/content.*.write`
 - Added scribable class dependency.


## v4.0.4 (2020-02-05)
------------------------

Fixes [#22](https://github.com/blake-regalia/graphy.js/issues/22), a bug when validating IRIs in the N-Triples and N-Quads readers.

### 🔧 Fixes

#### `@graphy/content.n*.read`
 - IRI validation


## v4.0.3 (2020-01-07)
------------------------

Patch for negative lookbehind RegExp polyfills **in all readers** that affects certain browsers including Firefox.

### 🔧 Fixes

#### `@graphy/content.*.read`
 - Lookbehind RegExp polyfills


## v4.0.2 (2019-12-29)
------------------------

Turtle and Trig reader performance patch.

### ⚡︎ Performance

#### `@graphy/content.t*.read`
 - Faster lexing and validation for prefixed names.


## v4.0.1 (2019-12-21)
------------------------

Fixed tag selectors in quad filter expressions, and added a patch for reading large string literals.

### 🔧 Fixes

#### `graphy` CLI
 - Tag selectors have been fixed.

#### `@graphy/content.*.read`
 - Fixed reading very large string literals.


### ⚡︎ Performance

#### `@graphy/content.*.read`
 - Optimized string literal lexing.


## v4.0.0 (2019-12-11)
------------------------

This update brings many new features to all the packages, with some necessary breaking changes, several fixes to the readers and writers, and performance improvements across the board.

### BREAKING CHANGES

#### `@graphy/core.data.factory`  **AFFECTS ALL PACKAGES**
 - `Term#equals()` and `Quad#equals()` now return `false` for falsy `other` arguments instead of throwing an Error per [RDFJS discussion](https://github.com/rdfjs/data-model-spec/issues/132).
 - IRI sanitization was previously performed on NamedNodes during uptake and output using `uri-js` from npm. This had some issues as it would sometimes modify IRIs to make them valid URIs. The sanitization process has been replaced with a simple Unicode character escaping for invalid sequences such as serializing spaces in IRIs as `'\u0020'`.

#### `@graphy/content.*.read`
 - Validation is now enabled by default. The `.validate` option to [ReadConfig](https://graphy.link/content.textual#config_read) has been deprecated. A new `.relax` option has been added which will allow users to bypass validation for faster reads.
 - The `.maxStringLength` option to [ReadConfig](https://graphy.link/content.textual#config_read) is now set to `Infinity` by default.

#### `@graphy/content.*.write`
 - The `.tokens.graph` option to [WriteConfig](https://graphy.link/content.textual#config_write) has been moved to `.style.graphKeyword`.
 - The previously undocumented `.collections` option to [WriteConfig](https://graphy.link/content.textual#config_write) has been renamed to `.lists` and is now officially supported.

#### `@graphy/util.dataset.tree` => `@graphy/memory.dataset.fast`
 - This package has been renamed to better align in with new similar packages in upcoming minor releases. Requires to the old package will still work in this release but will issue a deprecation warning.


#### CLI (Command-Line Interface)
 - All commands have been changed:
   - ~~`content.nt.read`~~ --> `read -c nt`
   - ~~`content.nq.read`~~ --> `read -c nq`
   - ~~`content.ttl.read`~~ --> `read -c ttl`
   - ~~`content.trig.read`~~ --> `read -c trig`
   - ~~`content.nt.write`~~ --> `write -c nt`
   - ~~`content.nq.write`~~ --> `write -c nq`
   - ~~`content.ttl.write`~~ --> `write -c ttl`
   - ~~`content.trig.write`~~ --> `write -c trig`
   - ~~`util.dataset.tree`~~ --> `tree`, `union`, `diff`, `minus`, `intersect`, `equals`, `contains` or `disjoint`


### Added Features

#### New packages
The following packages are content *scribers*, for serializing RDF fast. They are essentially the high-performance versions of the content writers with less features.
 - `@graphy/content.nt.scribe`
 - `@graphy/content.nq.scribe`
 - `@graphy/content.ttl.scribe`
 - `@graphy/content.trig.scribe`

#### New WritableDataEvents
[WritableDataEvents](#https://graphy.link/content.textual#interface_writable-data-event) are rich object structures that are capable of passing entire trees of RDF data or miscellaneous serialization directives such as prefix statements, comments, etc., to serializers. The following types were added:
 - `'c3r'` and `'c4r'` -- the 'strict-mode' variant of [concise triples and concise quads hashes (`'c3'` and `'c4'`)](https://graphy.link/concise#hash_c4), mostly used internally for passing data quickly between packages and for serializing with less safety check overhead.
 - `'comment'` and `'newlines'` -- added as a standlone data event types to allow for serializing comments/newlines to *scribers* whereas previously with the writers you could simply include factory directives inline the 	`'c3'` and `'c4'` events.

#### `@graphy/content.*.write`
 - Added more control over stylized output via the `.style` config option, which includes indentation control, keyword selection, optional block omission, and custom list serialization.
 - The `data` argument to `Writer#write(data)` now accepts plain Quads (scribers also accept plain Quads).
 - Added `Writer#import()` method for compatibility with [@RDFJS/Sink](http://rdf.js.org/stream-spec/#sink-interface).

#### `@graphy/content.*.read`
 - Added support for `.dataFactory` option to [ReadConfig](https://graphy.link/content.textual#config_read-no-input) for compatibility with [@RDFJS/ConstructorOptions](http://rdf.js.org/stream-spec/#constructoroptions-interface).

#### `@graphy/core.data.factory`
 - `factory.comment()` -- added `.width` option to support word-wrapped comments.
 - Specialized RDF Literal classes distinguish between simple, languaged, and datatyped literals with `.isSimple`, `.isLanguaged` and `.isDatatyped` prototype properties.
 - Added support for the [@RDFJS/Variable term type](http://rdf.js.org/data-model-spec/#variable-interface) via `factory.variable()`.


#### CLI (Command-Line Interface)
Many new features have been added to the CLI.
 - Added the `'/'` character to alias the internal pipe delimiter for a much cleaner syntax (`--pipe` is still supported).
 - **New Commands:**
   - `skip`, `head`, and `tail` for limiting the size of data that goes thru.
   - `filter` allows for [filtering quads via arbitrary constraints](https://graphy.link/quad-filter-expressions).
   - `transform` allows for applying quick and concise JavaScript functions from the CLI to modify Quads on-the-fly as they pass thru.
   - `concat` and `merge` reduce multiple input streams into a single output stream.
   - `read`, `write`, and `scribe` to replace package name commands.
   - `tree`, `union`, `diff`, `minus`, `intersect`, `equals`, `contains` and `disjoint` to replace dataset commands (all the latter of which canoncialize inputs by default).


## v3.2.2 (2019-02-09)
------------------------

#### Removed Dependencies From `graphy`

  The build script was accidentally assigning devDependencies from the super-repo to the `graphy` package, adding unnecessary bulk to the package install; those dependencies have been removed.


## v3.2.1 (2019-02-07)
------------------------

#### Error Message Fix

  Fixed content writers accessing property on undefined while trying to throw Error.


## v3.2.0 (2019-01-24)
------------------------

#### Add Quad Component Filters for Content Readers

  The CLI now supports filtering quads by specific components (i.e., subject, predicate, object or graph). Allows for selecting those that have a specific value for a given component, those that do not have any of the specific values for a given component (i.e., a negative filter list), or a combination of the two (e.g., quads that have this specific subject and do not have these specific predicates).


## v3.1.1 (2019-01-18)
------------------------

#### Fix Missing CLI Dependency and Patch for Browser Stream Polyfill

  The CLI `package.json` was missing a dependency to `yargs`. The [`stream` polyfill](https://github.com/browserify/stream-browserify) used by both Webpack and Browserify improperly pushes `null` and emits an `'end'` event during the call to `.destroy()` -- the same bug exists for node.js <v10. This patch includes an overriding destroy implementation for the stream polyfill when bundled.


## v3.1.0 (2019-01-16)
------------------------

#### Textual Content Writers

  Introduces textual content writers which accept writable data events such as concise-quads and concise-triples hashes, prefix mappings, quad objects, and so on. This update also includes several patches for content reader corner cases that were not covered by W3C manifest, as well as some fixes to situational stream handling and dataset tree.


## v3.0.6 (2018-12-30)
------------------------

#### Turtle and TriG reader patches

  Patches a blank node regex to cover a corner-case in reading Turtle and TriG documents.


## v3.0.5 (2018-12-21)
------------------------

#### Build Process Fix

  Due to how unit test manifests are cached, the build process needed a patch to ensure Travis CI can carry out test validation properly.


## v3.0.4 (2018-12-21)
------------------------

#### Comprehensive Unit Testing for Content Readers

  Content readers for N-Triples, N-Quads, Turtle and TriG now pass all unit tests provided by W3C when validation is enabled.


## v3.0.3 (2018-12-14)
------------------------

#### Core Data Factory Patch

  Since the initial release was a bit rushed, missed a latent dependency added to the `core.data.factory` package.


## v3.0.2 (2018-12-13)
------------------------

#### Passing Specification Tests

  The main four textual content readers, N-Triples, N-Quads, Turtle and TriG now pass all specification tests. More unit tests needed to cover all API use cases.


## v3.0.1 (2018-12-11)
------------------------

#### Functional Readers

  A complement to the initial beta release; adds content readers and patches an issue in the `core.data.factory` package without complete unit tests yet.


## v3.0.0 (2018-12-10)
------------------------

#### Beta Production Release

  Had to get out a beta on npm before unit tests could be completed due to pressure over the temporarily dormant `graphy` org name on npm.
