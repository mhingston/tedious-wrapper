# tedious-wrapper

A simple wrapper for [tedious](http://github.com/tediousjs/tedious) (TDS driver) with a built-in [connection pool](https://github.com/mhingston/tedious-connection-pool).

## Installation

    npm install mhingston/tedious-wrapper
    
## Description
The API thinly wraps the `bulkLoad`, `execSql` and `callProcedure` methods of the tedious [Connection](http://tediousjs.github.io/tedious/api-connection.html) object.

## Instantiation

```javascript
const TediousWrapper = require('tedious-wrapper');
const config = // example config
{
    connection:
    {
        userName: 'user',
        password: 'pass',
        server: 'localhost',
        port: 1433,
        options:
        {
            abortTransactionOnError: true,
            database: 'master',
            connectTimeout: 15000,
            requestTimeout: 120000,
            encrypt: true,
            isolationLevel: 'SNAPSHOT',
            useUTC: false
        }
    },
    pool:
    {
        min: 1,
        max: 10,
        idleTimeout: 10000,
        acquireTimeout: 12000,
        log: false
    }
}

const db = new TediousWrapper(config, logger);
```

## Class: `TediousWrapper`

`const db = new TediousWrapper(config, logger);`

* `config` {Object} the configuration object. The `connection` property uses the same configuration options as the tedious' [Connection](http://tediousjs.github.io/tedious/api-connection.html#function_newConnection) object. The `pool` property uses the same configuration options as [tedious-connection-pool](https://github.com/mhingston/tedious-connection-pool#new-connectionpoolpoolconfig-connectionconfig)'s `poolConfig`.
* `logger` {Boolean|Function} Set to true to have debug log written to the console or pass a function to receive the log messages. Default = `false`.

### Property: `types`

Returns tedious [TYPES](http://tediousjs.github.io/tedious/api-datatypes.html).

`db.types;`

### Method: `connect()`

Acquire a Tedious Connection object from the pool. This method returns a promise, but you can use a callback function if you prefer.

 * `callback(error, connection)` {Function} Callback function.
   * `error` {Object} Object containing an error that occurred whilst trying to acquire a connection, otherwise null.
    * `connection` {Object} A [Connection](https://tediousjs.github.io/tedious/api-connection.html).

```javascript
// Callback
db.connect((error, connection) =>
{
    // your code goes here
});

// Promise
db.connect()
.then((connection) =>
{
    // your code goes here
})
.catch((error) =>
{
    // your code goes here
});

// Async/await
async function connect()
{
    let connection;

    try
    {
        connection = await db.connect();
    }

    catch(error)
    {
        // your code goes here
    }
};
```

### Method: `bulkLoad({table, columns, rows, callback})`

Perform a [bulkLoad](http://tediousjs.github.io/tedious/bulk-load.html). This method returns a promise, but you can use a callback function if you prefer.

 * `table` {String} Name of the table.
 * `columns` {Array[]} An array of arrays containing the the [column definitions](http://tediousjs.github.io/tedious/bulk-load.html#function_addColumn).
 * `rows` {Array[]} An array of [   columnArray](http://tediousjs.github.io/tedious/bulk-load.html#function_addRow).
 * `callback(error, rowCount)` {Function} Callback function.
    * `error` {Object} Object containing an error that occurred whilst trying to use bulkLoad.
    * `rowCount` {Number} Number of rows inserted.

```javascript
const myBulkLoad =
{
    table: 'MyTable',
    columns:
    [
        ['MyIntColumn', db.types.Int, {nullable: false}],
        ['MyVarCharColumn', db.types.VarChar, {nullable: false}]
    ],
    rows:
    [
        [1, 'A'],
        [2, 'B'],
        [3, 'C']
    ],
    callback((error, rowCount) =>
    {
        // your code goes here
    })
}

// Callback
db.bulkLoad(myBulkLoad);

// Promise
db.bulkLoad(myBulkLoad)
.then((rowCount) =>
{
    
})
.catch((error) =>
{

});

// Async/await
async function bulkLoad()
{
    let rowCount;

    try
    {
        rowCount = await db.bulkLoad(myBulkLoad);
    }

    catch(error)
    {

    }
};
```

### Method: `request({sql, parameters, options, callback})`

Perform a [request](http://tediousjs.github.io/tedious/api-request.html). This method returns a promise, but you can use a callback function if you prefer.

 * `sql` {String} The SQL statement to be executed (or a procedure name, if the request is to be used with `callProcedure`).
 * `parameters` {Array[]} An array of arrays containing the the [parameter definitions](http://tediousjs.github.io/tedious/api-request.html#function_addParameter).
 * `options` {Object} A object containing the below properties.
   * callProcedure {Boolean} Use [callProcedure](http://tediousjs.github.io/tedious/api-connection.html#function_callProcedure).
   * transformers {Object[]} An array of objects as below. This is used for defining rules for transforming raw values into other representations as required, e.g. formatting date objects as strings. If you define multiple rules that affect the same column(s) only the first rule will be processed.

   ```javascript
   {
       column: {String|String[]|RegExp}, // column name(s) to match
       resultSets: {Number[]}, // result sets (zero-indexed) to match. If omitted the columnName mapping above will apply to all result sets. 
       transform: {Function(value, metadata)} // A function that takes the input value and returns an output value
   }
   ```

 * `callback(error, resultSets)` {Function} Callback function.
    * `error` {Object} Object containing an error that occurred whilst trying to use request.
    * `resultSets` {Array[]} An array of arrays containing the result sets from the request.

```javascript
const myRequest =
{
    sql: 'SELECT FirstName, LastName, EmailAddress FROM [User] WHERE UserID = @UserID',
    parameters:
    [
        ['UserID', db.types.Int, 1]
    ],
    callback((error, resultSets) =>
    {
        // your code goes here
    })
}

// Callback
db.request(myRequest);

// Promise
db.request(myRequest)
.then((resultSets) =>
{
    
})
.catch((error) =>
{

});

// Async/await
async function request()
{
    let resultSets;

    try
    {
        resultSets = await db.request(myRequest);
    }

    catch(error)
    {

    }
};
```

### Method: `destroy()`

Closes the connection pool associated to the `TediousWrapper` instance. Once the connection pool has been closed no new connections can be opened.