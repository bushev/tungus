Tungus
======

[![Build Status](https://travis-ci.org/sergeyksv/tungus.png?branch=master)](https://travis-ci.org/sergeyksv/tungus)

__Note! This version of the driver only works with Mongoose >= 4.x. It might not be backwards compatible with Tungus 0.0.x databases yet, as it uses native ObjectIDs.__

This module implements mongoose.js driver API and allows to use mongoose with [TingoDB](http://www.tingodb.com).

TingoDB is embedded Node.js database that is compatible with MongoDB on API level.

So far this module is on its early stage with only basic functionality.

To use this module you have to install both tungus and mongoose.

	npm install tungus
	npm install mongoose

Then in your code you should include once tungus module prior to include of mongoose.
This rewrites global.MONGOOSE_DRIVER_PATH variable to point it to tungus.

```javascript
require('tungus');
require('mongoose');
```

Next to that you can keep using mongoose as usual except now it will accept different connection string:

```javascript
mongoose.connect('tingodb:///some/local/folder');
```

Optionally you can set tingodb options using ```TUNGUS_DB_OPTIONS``` global variable. For example this way it is possible to switch to BSON.ObjectID ids which is default for mongodb.

```javascript
global.TUNGUS_DB_OPTIONS = { nativeObjectID: true, searchInArray: true };
```

Full example:
```javascript
const tungus = require('tungus');
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

console.info('Running mongoose version %s', mongoose.version);

/**
 * Console schema and model
 */

const consoleSchema = Schema({
  name: String,
  manufacturer: String,
  released: Date
});
const Console = mongoose.model('Console', consoleSchema);

/**
 * Game schema and model
 */

const gameSchema = Schema({
  name: String,
  developer: String,
  released: Date,
  consoles: [{ type: Schema.Types.ObjectId, ref: 'Console' }]
});
const Game = mongoose.model('Game', gameSchema);

/**
 * Connect to the local tingo db file (NOT using MongoClient)
 */

mongoose.Promise = global.Promise;
mongoose.connect(`mongodb://data/base`, { useMongoClient: false })
  // Create data (console)
  .then(() => Console.create({
    name: 'Nintendo 64',
    manufacturer: 'Nintendo',
    released: 'September 29, 1996'
  }))
  // Create data (game)
  .then((nintendo64) => Game.create({
    name: 'Legend of Zelda: Ocarina of Time',
    developer: 'Nintendo',
    released: new Date('November 21, 1998'),
    consoles: [nintendo64]
  }))
  // Run example
  .then(example)
  // Something bad happened!
  .catch((err) => console.error('Example failed!', err));

/**
 * Example: Population
 */

function example () {
  return Game
    .findOne({ name: /^Legend of Zelda/ })
    .populate('consoles')
    .then((ocarina) => {
      console.log(ocarina);
      console.log(
        '"%s" was released for the %s on %s',
        ocarina.name,
        ocarina.consoles[0].name,
        ocarina.released.toLocaleDateString()
      );
    })
    .then(cleanup);
}

/**
 * Remove elements and disconnect
 */

function cleanup () {
  return Promise.resolve()
    .then(() => Console.remove({}))
    .then(() => Game.remove({}))
    .then(() => mongoose.disconnect());
}
```
