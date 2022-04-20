### BDD framework training. Built-in utils

[![N|Solid](https://images.ctfassets.net/fikanzmkdlqn/5NoHRB1q6lrNzSSpekhrG5/cf22f3d7d9e82aed5e79659800458b57/TELUS_TAGLINE_HORIZONTAL_EN.svg)](https://www.telus.com/en/)

##### [Framework documentation](https://github.com/telus/telus-bdd-docs)

In this section you will discover usefull utils built into the framework.

<img src="https://cdn4.iconfinder.com/data/icons/48-bubbles/48/23.Videos-512.png" width="30px" margin-top="15px"/> [Session video recording](https://drive.google.com/file/d/1TeUHsnBO7655zaV_X3BRvVdBEOVCx4W9/view?usp=sharing)

[<img src="https://cdn-icons-png.flaticon.com/512/25/25231.png" width="30px"> Code Example](https://github.com/telus/bdd-demo/blob/ee275b27de6e576105cd0fe15e57bf32c5807049/bdd/steps/PI.steps.ts#L81)

#### **Contents:**

1. [postgresQueryExecutor database query util](#postgresqueryexecutor-database-query-util)
2. [validateObject json schema validator](#validateobject-json-schema-validator)

#### **postgresQueryExecutor database query util**
BDD framework has a built in util for making SQL queries to PostgreSQL databases.
1. Import util from the framework:
```typescript
const {
  postgresQueryExecutor,
} = require("@telus-bdd/telus-bdd");
```
2. Call it inside of some step callback function:
```typescript
and(/^finish testing for (.*)$/, async (id: number) => {
  await postgresQueryExecutor(unsetTestingFlag(id));
  console.log(`Finished testing for dataset ${id}`);
});
```
It is that simple, util takes in one required and two optional parameters:
- connection config
- sql query string (required)
- sql query params

By default query executor makes queries to the database specified in [`bdd.config` file](./framework-intro.md/#bddconfig-file). But if you need to query other PostgreSQL databases, you just need to pass a config object as the first parameter:
```typescript
and(/^finish testing for (.*)$/, async (id: number) => {
  const config = {
      user: 'user',
      password: 'password',
      host: 'localhos',
      port: '5432',
      database: 'db1'
  };  
  await postgresQueryExecutor(config, unsetTestingFlag(id));
  console.log(`Finished testing for dataset ${id}`);
});
```

Also if your query is big in size and the values used in a query are variables, you can use the 3rd argument in query executor, which is query parameters array, but for this query string should also follow specific format (all of the syntaxes can be found by this [link](https://node-postgres.com/features/queries)):
```typescript
and(/^finish testing for (.*)$/, async (id: number) => {
  const config = {
      user: 'user',
      password: 'password',
      host: 'localhos',
      port: '5432',
      database: 'db1'
  };  
  await postgresQueryExecutor(config, 'insert into table (id, name, value) values (:id, :name, :value)', {id, name, value});
  console.log(`Finished testing for dataset ${id}`);
});
```
#### **validateObject json schema validator**
Another usefull util BDD framework has, it can validate json objects(e.g. requests, responses) against JSON Schemas. 

In order to provide the highest performance it integrates with the best JSON schema validator library [AJV](https://ajv.js.org/) it has a big variaty of available validations that you can find in the original [docs](https://ajv.js.org/json-schema.html).

In order to use validation you only need to do 3 simple steps:

1. Add a line to your [`bdd.config.js/json` file](./framework-intro.md/#bddconfig-file) to specify the path to schemas directory and create this directory:

```javascript
{
  schemaPath: "./bdd/schema";
}
```

2. Create a `${scemaName}.json` file with AJV format JSON Schema (e.g.):

```json
{
  "response": {
    "type": "object",
    "properties": {
      "foo": { "type": "string" },
      "bar": { "type": "number" }
    },
    "required": ["foo"]
  }
}
```

3. Import a validateObject util from the framework and pass object with `${schemaName}` name of the schema file without extension to it:

```javascript
const { validateObject } = require("@telus-bdd/telus-bdd");
//...
const isValid = validateObject(response, schemaName);
//...
```

Validator will return a boolean value and in case of any errors provide a message and schema path to the error element.
