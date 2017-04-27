# Fabricate and delete your data on integration testing.

## What's it?
`db-fabricator` is a simple node.js module that can create, update and delete data and also manipulate created and updated data during sessions.

## Simple Example

```javascript
const fab = require('db-fabricate');
const MySQLAdapter = fab.MySQLAdapter;
const Fabricator = new fab.Fabricator(new MySQLAdapter({database: 'TestDatabase'})); // CHANGE YOUR DB CREDENTIALS HERE

Fabricator.startSession();

Fabricator.create('TestTable', {
  name: '123'
}).then((id) => {
  // here we have element we have created.
  return Fabricator.stopSession();
}).then(() => {
  // stop session killed all created elements during the session.
  Fabricator.closeConnection();
});
```

**NOTE:** Defaults for database connection are:
```
{
  host: 'localhost',
  user: 'root',
  password: ''
}
```


This also works with updates:

```javascript
// ...
Fabricator.update('TestTable', {
  name: '123'
}, {
  id: 1
}).then(() => {
  // here element with id = 1 will have name = '123'
  return Fabricator.stopSession();
}).then(() => {
  // stop session will restore defaults for column with id = 1
  Fabricator.closeConnection();
});
```

So you can easily create integration tests with on-the-go data and simply remove it after creation.

# Documentation for methods:

## Fabricator.create()

Will create new entity in database with passed parameters:

```javascript
Fabricator.create('TestTable', { // will be written into Session 2
  name: '123'
}).then((id) => {
  console.log(id); // id of created entity
});
```

## Fabricator.startSession()

Starts new session for fabricator. You can nest sessions. Data will be recoreded to the latest opened session.
On close session all data created or modified in this session will be restored.

```javascript
Fabricator.startSession(); // Session 1
Fabricator.startSession(); // Session 2

Fabricator.create('TestTable', { // will be written into Session 2
  name: '123'
}).then((id) => {
  // do some tests...

  return Fabricator.closeSession(); // element with name 123 will be removed and Session 2 will be closed.
}).then(() => {
  return Fabricator.create('TestTable', { // will be written into Session 1
    name: '321'
  });
}).then(() => {
  return Fabricator.closeSession(); // element with name 321 will be removed. and Session 1 will be closed.
});

```

## Fabricator.stopSession()

Stops latest opened session.
On close session all data created or modified in this session will be restored.

```javascript
Fabricator.startSession();
Fabricator.closeSession().then(() => {
  console.log('session closed');
});
```

## Fabricator.update(table, data, constraints)

Updates entity/enitities in `table` with fields and values specified in `data`. Entities will be found by passed constraints.

```javascript
Fabricator.startSession();
Fabricator.update('TestTable', {name: '123'}, {id: 1}).then(() => {
  // do some tests...

  return Fabricator.closeSession();
}).then(() => {
  console.log('session closed');
});
```

## Fabricator.delete(table, data)

Removes data from `table` by passed list of ids (or single id):
*NOTE:* Removed data wont be restored on session close. _To be implemented_

```javascript
Fabricator.remove('TestTable', [1]).then(() => {
  // do some tests...
});
```

## Fabricator.closeConnection()

Closes connection to current database.

```javascript
Fabricator.closeConnection();
console.log('connection closed');
```