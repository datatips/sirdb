# :man: [Sir](https://github.com/c9fe/sirdb) ![npm downloads](https://img.shields.io/npm/dt/stubdb) ![version](https://img.shields.io/npm/v/stubdb?label=version)

**Sir.DB** -- A very simple database on the file system, using JSON files organised into subdirectories for each table. 



**Around 500 source lines of code** (see stats folder)

## features

- Human readable
- Store any JSON-able object
- Index on any property (for nested objects, only index top-level properties)
- Auto ID or custom ID
- Diffable by git 
- All records and indexes and table information is just JSON files
- 1 file per record, 1 file per unique index value, 1 file per table info
- 1 sub-directory per table, 1 sub-directory (nested inside table) per indexed property

### Roadmap

- Transactions
- ACID guarantee (as long as accessed by single node thread)
- Can expand ACID to multi-threaded access with a request queue.

All in all this makes the database easy to understand and inspect. As well as making the code easy to read and maintain.

## get

```console
npm i --save sirdb
```

## api

There's only a couple of handful of calls in this api: `config`, `dropTable`, `getIndexedTable`, `getTable`, `put`, `get`, `getAllMatchingKeysFromIndex` and `getAllMatchingRecordsFromIndex`

## config({root: path})

Configure the root directory of your database.

## dropTable(name: string)

Deletes a table.

## getTable(name: string): Table 

Creates a table (if it doesn't exist) and returns it.

## getIndexedTable(name: string, indexed_properties: string[]): IndexedTable

Creates a table (if it doesn't exist) that has indexes for the given properties, and returns it.

## &lt;Table&gt;.put(key: string, value: any, greenlights?: function | function[] | Evaluator)

Adds (or updates) an item to the table by key.

## &lt;Table&gt;.get(string: key, greenlights?: function | function[] | Evaluator)

Gets an item from the table by key.

## &lt;IndexedTable&gt;.getAllMatchingKeysFromIndex(prop: string, value: any): string[]

Gets all item keys from the table that have a property `prop` that matches `value`, if that property is indexed.

## &lt;IndexedTable&gt;.getAllMatchingRecordsFromIndex(prop: string, value: any): any[]

Gets all items from the table that have a property `prop` that matches `value`, if that property is indexed.

## &lt;Table&gt;.getAll(): any[]

Gets all items from the table.

## &lt;PageableTable&gt;.getPage(cursor: string, count?: int): any[]

Get `count` (default 10) items from the table, starting at the first item after `cursor`. *Note: not yet implemented.*

## Example

See below for how they're used:

```javascript
import path from 'path';
import assert from 'assert';
import fs from 'fs';

import {getTable, getIndexedTable, dropTable, config} from './api.js';

testGetTable();
testGetIndexedTable();
testDropTable();
testPut();
testIndexedPut();
testGet();
testGetAll();

function testGetTable() {
  const root = path.resolve(__dirname, "test-get-table");
  config({root});

  dropTable("users");
  dropTable("subscriptions");

  getTable("users");
  getTable("subscriptions");

  assert.strictEqual(fs.existsSync(path.resolve(root, "users", "tableInfo.json")), true);
  assert.strictEqual(fs.existsSync(path.resolve(root, "subscriptions", "tableInfo.json")), true);

  fs.rmdirSync(root, {recursive:true});
}

function testGetIndexedTable() {
  const root = path.resolve(__dirname, "test-get-indexed-table");
  config({root});

  dropTable("users");
  dropTable("subscriptions");

  getIndexedTable("users", ["username", "email"]);
  getIndexedTable("subscriptions", ["email"]);

  try {
    assert.strictEqual(fs.existsSync(path.resolve(root, "users", "tableInfo.json")), true);
    assert.strictEqual(fs.existsSync(path.resolve(root, "subscriptions", "tableInfo.json")), true);

    assert.strictEqual(fs.existsSync(path.resolve(root, "users", "_indexes", "username", "indexInfo.json")), true);
    assert.strictEqual(fs.existsSync(path.resolve(root, "users", "_indexes", "email", "indexInfo.json")), true);
    assert.strictEqual(fs.existsSync(path.resolve(root, "subscriptions", "_indexes", "email", "indexInfo.json")), true);
  } catch(e) {
    console.error(e);
  }

  fs.rmdirSync(root, {recursive:true});
}

function testDropTable() {
  const root = path.resolve(__dirname, "test-drop-table");
  config({root});

  getTable("users");
  getTable("subscriptions");

  dropTable("users");
  dropTable("subscriptions");

  assert.strictEqual(fs.existsSync(path.resolve(root, "users", "tableInfo.json")), false);
  assert.strictEqual(fs.existsSync(path.resolve(root, "subscriptions", "tableInfo.json")), false);

  fs.rmdirSync(root, {recursive:true});
}

function testPut() {
  const root = path.resolve(__dirname, "test-get-table");
  const errors = [];
  let gotItems;
  let key = 1;

  config({root});

  dropTable("items");

  const Items = getTable("items");
  const items = [
    {
      key: `key${key++}`,
      name: 'Apple',
      type: 'fruit',
      grams: 325
    },
    {
      key: `key${key++}`,
      name: 'Pear',
      type: 'fruit',
      grams: 410
    },
    {
      key: `key${key++}`,
      name: 'Soledado',
      type: 'career',
      grams: null,
      qualities_of_winners: [
        "devisiveness",
        "rationality",
        "aggression",
        "calmness"
      ]
    },
  ];
  
  try {
    items.forEach(item => Items.put(item.key, item));
  } catch(e) {
    errors.push(e);
  }

  assert.strictEqual(errors.length, 0);

  try {
    gotItems = items.map(item => Items.get(item.key));
  } catch(e) {
    errors.push(e);
  }

  assert.strictEqual(errors.length, 0);

  assert.strictEqual(JSON.stringify(items), JSON.stringify(gotItems));

  fs.rmdirSync(root, {recursive:true});
}

function testIndexedPut() {
  const root = path.resolve(__dirname, "test-indexed-put-table");
  const errors = [];
  let gotItems1 = [], gotItems2 = [], gotItems3 = [];
  let key = 1;

  config({root});

  dropTable("items");

  const Items = getIndexedTable("items", ["name", "type"]);
  const items = [
    {
      key: `key${key++}`,
      name: 'Apple',
      type: 'fruit',
      grams: 325
    },
    {
      key: `key${key++}`,
      name: 'Pear',
      type: 'fruit',
      grams: 410
    },
    {
      key: `key${key++}`,
      name: 'Soledado',
      type: 'career',
      grams: null,
      qualities_of_winners: [
        "devisiveness",
        "rationality",
        "aggression",
        "calmness"
      ]
    },
  ];
  
  try {
    try {
      items.forEach(item => Items.put(item.key, item));
    } catch(e) {
      errors.push(e);
    }

    assert.strictEqual(errors.length, 0);

    try {
      gotItems1 = items.map(item => Items.get(item.key));
    } catch(e) {
      errors.push(e);
    }

    assert.strictEqual(errors.length, 0);

    assert.strictEqual(JSON.stringify(items), JSON.stringify(gotItems1));

    try {
      gotItems2 = Items.getAllMatchingRecordsFromIndex('name', 'Apple');
    } catch(e) {
      errors.push(e);
    }

    assert.strictEqual(errors.length, 0);
    assert.strictEqual(gotItems2.length, 1);
    assert.strictEqual(JSON.stringify(items.slice(0,1)), JSON.stringify(gotItems2.map(([,val]) => val)));

    try {
      gotItems3 = Items.getAllMatchingRecordsFromIndex('type', 'fruit');
    } catch(e) {
      errors.push(e);
    }

    assert.strictEqual(errors.length, 0);
    assert.strictEqual(gotItems3.length, 2);
    assert.strictEqual(JSON.stringify(items.slice(0,2)), JSON.stringify(gotItems3.map(([,val]) => val)));

  } catch(e) {
    console.error(e);
    console.log(errors);
  }

  fs.rmdirSync(root, {recursive:true});
}

function testGet() {
  const root = path.resolve(__dirname, "test-get-table");
  const errors = [];
  const gotItems = [];
  let key = 1;

  config({root});

  dropTable("items");

  const Items = getTable("items");
  const items = [
    {
      key: `key${key++}`,
      name: 'Apple',
      type: 'fruit',
      grams: 325
    },
    {
      key: `key${key++}`,
      name: 'Pear',
      type: 'fruit',
      grams: 410
    },
    {
      key: `key${key++}`,
      name: 'Soledado',
      type: 'career',
      grams: null,
      qualities_of_winners: [
        "devisiveness",
        "rationality",
        "aggression",
        "calmness"
      ]
    },
  ];
  
  for( const item of items ) {
    try {
      gotItems.push(Items.get(item.key));
    } catch(e) {
      errors.push(e);
    }
  }

  assert.strictEqual(errors.length, 3);
  assert.strictEqual(gotItems.length, 0);

  fs.rmdirSync(root, {recursive:true});
}

function testGetAll() {
  const root = path.resolve(__dirname, "test-get-table");
  const errors = [];
  let gotItems;
  let key = 1;

  config({root});

  dropTable("items");

  const Items = getTable("items");
  const items = [
    {
      key: `key${key++}`,
      name: 'Apple',
      type: 'fruit',
      grams: 325
    },
    {
      key: `key${key++}`,
      name: 'Pear',
      type: 'fruit',
      grams: 410
    },
    {
      key: `key${key++}`,
      name: 'Soledado',
      type: 'career',
      grams: null,
      qualities_of_winners: [
        "devisiveness",
        "rationality",
        "aggression",
        "calmness"
      ]
    },
  ];
  
  try {
    items.forEach(item => Items.put(item.key, item));
  } catch(e) {
    errors.push(e);
  }

  assert.strictEqual(errors.length, 0);

  try {
    gotItems = Items.getAll();
  } catch(e) {
    errors.push(e);
  }

  assert.strictEqual(errors.length, 0);

  items.sort((a,b) => a.key < b.key ? -1 : 1);
  gotItems.sort((a,b) => a.key < b.key ? -1 : 1);
  assert.strictEqual(JSON.stringify(items), JSON.stringify(gotItems));

  fs.rmdirSync(root, {recursive:true});
}
 ```

-------------

# *Sir!*

