# random-access-corestore
[![Build Status](https://travis-ci.com/andrewosh/random-access-corestore.svg?token=WgJmQm3Kc6qzq1pzYrkx&branch=master)](https://travis-ci.com/andrewosh/random-access-corestore)

A simple corestore that wraps a random-access-storage module. This module is the canonical implementation of the "corestore" interface, which exposes a hypercore factory and a set of associated functions for managing generated hypercores.

Corestore imposes the convention that the first requested hypercore defines the discovery key (and encryption parameters) for its replication stream.

### Installation
`npm i random-access-corestore --save`

### Usage
A random-access-corestore instance can be constructed with either a random-access-storage module, or a function that returns a random-access-storage module given a path:
```js
const corestore = require('random-access-corestore')
const ram = require('random-access-memory')
const raf = require('random-access-file')
const store1 = corestore(ram)
const store2 = corestore(path => raf('store2/' + path))
```

Hypercores can be generated with `get`. The first core that's generated will be considered the "main" core, and will define the corestore's replication parameters:
```js
const core1 = store1.get()
```

Additional hypercores can be created either by key or by name. If a hypercore is created by name, it will be stored as such in the storage layer (e.g. in the `second` directory). Named hypercores are useful when instantiating a new hypercore-based data structure and the hypercore keys have not yet been generated:
```js
const core2 = store1.get({ name: 'second' })
```
Once a named hypercore has been instantiated, it's indexed by both its key and its discovery key in memory, so that it can be injected into replication streams.

Two corestores can be replicated with the `replicate` function, which accepts hypercore's `replicate` options:
```js
const store2 = corestore(ram)
const core3 = store2.get(core1.key)
const stream = store1.replicate()
stream.pipe(store2.replicate()).pipe(stream)
```

### API
#### `const store = corestore(storage)`
Create a new corestore instance. `storage` can be either a random-access-storage module, or a function that takes a path and returns a random-access-storage instance.

#### `store.get(opts)`
Create a new hypercore. Options can be one of the following:
```js
{
  key: 0x1232..., // A Buffer representing a hypercore key
  discoveryKey: 0x1232... // A Buffer representing a hypercore discovery key (must have been previously created by key)
  name: 'core-name', // A name for the new hypercore, if the key has not yet been generated
  ...opts // All other options accepted by the hypercore constructor
}
```

If `opts` is a Buffer, it will be interpreted as a hypercore key.

#### `store.on('feed', feed, options)`

Emitted everytime a feed is loaded internally (ie, the first time get(key) is called).
Options will be the full options map passed to .get.

#### `store.replicate(opts)`
Create a replication stream for all generated hypercores. The stream's handshake parameters (i.e. its discovery key) will be defined by the first hypercore created by the corestore.

`opts` can be any hypercore replication options.

#### `store.close(cb)`
Close all hypercores previously generated by the corestore.

### License
MIT
