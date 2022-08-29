### BDD framework training. Database Bootstrap

[![N|Solid](https://images.ctfassets.net/fikanzmkdlqn/5NoHRB1q6lrNzSSpekhrG5/cf22f3d7d9e82aed5e79659800458b57/TELUS_TAGLINE_HORIZONTAL_EN.svg)](https://www.telus.com/en/)

##### [Framework documentation](https://github.com/telus/telus-bdd-docs)

In this section you will learn what is database bootstrap and when it can be used.

<img src="https://cdn4.iconfinder.com/data/icons/48-bubbles/48/23.Videos-512.png" width="30px" margin-top="15px"/> [Session video recording]()

[<img src="https://cdn-icons-png.flaticon.com/512/25/25231.png" width="30px"> Code Example](https://github.com/telus/bdd-demo/blob/master/bdd/DBbootstrap.config.js)

#### Contents:

1. [Database bootstrap definition](#database-bootstrap-definition)
2. [Databse bootstrap in feature file](#databse-bootstrap-in-feature-file)
3. [SQL table and queries creation best practice](#sql-table-and-queries-creation-best-practice)
4. [DBBootstrap.config file](#dbbootstrapconfig-file)
5. [DBbootstrap types](#dbbootstrap-types)
6. [Tag parameters](#tag-parameters)
7. [DBbootstrap parametrization](#dbbootstrap-parametrization)

#### **Database bootstrap definition**
Database Bootstrap is a unique BDD framework mechanism, that allows to use data from SQL databases as parameters for steps in feature files.
You may have noticed the following parameters for steps:
```gherkin
Then response status code should be @status_code
```
Here `@status_code` is a parameter from DB provided by database bootstrap.

The mechanism is quite simple:
- User configures database bootstrap in a prescripted way
- User binds particular bootstrap configuration to a feature file
- When this feature file id launched, the framework runs an according to bound bootstrap SQL query and populates query result to the framework process environment
- This data is accessed in step definitions.

Database bootstrap allows to reuse only one feature file with any number of datasets stored in DB.

#### **Databse bootstrap in feature file**
As with any feature the usage of DBbootstrap starts from a feature file.
In order to use it you need to:
1. Specify a particular database bootstrap alias using according tag parameter
```gherkin
@DBbootstrap=testbootstrap
```
2. Add parameters from database to steps following the rules:
    - parameter name in feature file is always simillar to the name of a column in database where needed data is stored (for our example: there is a status_code column in the table, that contains expected status codes for datasets)
    - parameter name in feature file must start from `@` symbol 
```gherkin
Scenario: Make Product inventory call
    Given developer selected ecid and lpdsid
    When call TMF Product Inventory API
    Then response status code should be @status_code
```

When parameter starts with `@` the framework would automatically understand that this is a value from DB and in steps return to your callback function a real value of this parameter in DB, not `@status_code` string as it would happen with text parameters. So in terms of step implementation it looks the same for a developer, wether text or bootsrap parameter is used in a feature file.
You can also do feature file part configuration in the end, after, table in DB and config are ready.
#### **SQL table and queries creation best practice**
Let's create the most crucial part of database bootstrap, actually the database. 
>Note: As with any task there are multiple solutions, in database bootstrap there are multiple ways to create columns in your tables and write SQL requests (the next topic) and you are encouraged to use whichever way best suites your requirements. Here we will provide our best-practice for creating tables and queries for DB bootstrap for test automation case.

In our example, there is an API under test, so our table would contain according columns, except of that, such table should have a surrogate key `id` column and what is called "lock flag" column `testing` to prevent the dataset currently under test from being tested paralelly.

**Table bdd_datasets_demo:**

| id | ecid | lpdsid | testing | status\_code | schema |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | 99430809 | 4530740 | false | 200 | PISchema |

It is recommended to use surrogate keys as primary keys in all cases, because data in datasets may often repeat which will cause an error.

#### **DBBootstrap.config file**
Now when the table is ready, we need an SQL query that will pull data from the table. Such queries are configured in [`DBBootstrap.config file`](/framework-intro.md/#dbbootstrapconfig-file).

This config is a javascript file, that contains an exported object. Inside of this object there are database bootstrap queries configuration stored as key-value pairs. 
- Object **key** is the **alias** of bootstrap that is used to reference it in feaature files
- Object **value** is a function, receives `params` object with parameters ans returns a string of the SQL query. So value structure looks like this:
```javascript
(params) => `SELECT * FROM table LIMIT 1`
``` 
Following our best practice approach, an SQL query for our case should be created with `UPDATE RETURING` syntax, this syntax allows to update a row and get all its columns as a result (same as `SELECT` would do).
```javascript
module.exports = {
  "testbootstrap": (params) => 
  `UPDATE bdd_datasets_demo 
    SET testing=true 
    WHERE id IN (
      SELECT id FROM bdd_datasets_demo 
      WHERE not testing 
      LIMIT 1
    ) RETURNING *`
}
```
We will set "lock flag" to true, to indicate that a dataset is currently under test, to do this we would need to update a row in DB, but in the same time we need to get the data back, that's why the following syntax is used. It totally eliminates the possibility of the dataset being tested paralelly in several threads, or several servers connected to one DB even if requests come simultaneously (because as you can see by `WHERE` part, only not testing rows can be obtained and the set of "lock flag" happens immediately).

Also as you can see there is always only one row returned from the bootstrap (`LIMIT 1`). This helps to isolate the execution, each test and its dataset have dedicated process with own environmet with such approach. So we strongly recommend to do this and to write your steps logic to process one dataset at a time. 

> Note: Nevertheless, it is even possbile to obtain the whole table with one query and write steps, so they would process multiple parameter values.


**What would happen under the hood?**

Now when a feature would be launched:
- Framework would parse DBbootstrap alias from a feature file
- Framework would run a corresponding `DBBootstrap.config.js` method, using the alias to get text of an SQL query
- Framework would send this query to a connected in `bdd.config.js` DB
- Framework would populate query result data to process environment.
- Framework would parse feature file, recognize parameters with `@` at the start, take their real value from environemnt and pass it to step callback functions.

#### **DBbootstrap types**
There are two types of database bootstrap configuration
- The one we just used - raw query type
- And *object* type configuration, where you need to create a special format object to describe your bootstrap.

The first way is versitile, you can query the data any way you want. 

The second way is created to provide you a "framework" for queries, when you specify names of table and columns and the framework constructs a query for you. This approach is oriented on test-automation and migration-automation, you can read about it [here](/advanced-dataset-usage.md) in more details.
#### **Tag parameters**
Tag parameters are parameters used to provide some metadata about a feature file (called like this, because they start from a `@` symbol like a tag). Metadata related to DB bootstrap is provided with tag parameters. You have already faced `@DBbootstrap` tag, used to specify alias of DB bootstrap.

There are actually 3 of them related to bootstrap:
- `@DBbootstrap` - alias
- `@runTimes` - number of times feature file would be ran, in other words how many datasets would be processed (with the recommended approach one query - one row)
- `@DBbootstrapParams` - JSON object string (without any spaces!!!) of parameters that are passed to `params` argument of DB bootstrap function. 
By default framework puts two parameters into this argument:
    - `bddEnv` - value of passed environment, if specified (with `--bddEnv` argument in cli)
    - `runTimes` - value of `@runTimes` tag parameter, also could serve as CLI parameter
#### **DBbootstrap parametrization**
By specifing some more parameters in this tag, you can make your query dynamic, because `DBBootstrap.config` is `.js` file, so you can use any string methods available in javascript, for example:
```gherkin
@-
@-
@PIComponentTest
@DBbootstrap=testbootstrap
@DBbootstrapParams={"statusCode":"200"}
Feature: Test new db bootstrap with dataset id from cli
#...
```
```javascript
module.exports = {
  "testbootstrap": (params) => 
  `UPDATE bdd_datasets_demo 
    SET testing=true 
    WHERE id IN (
      SELECT id FROM bdd_datasets_demo 
      WHERE not testing
      AND env=${params.bddEnv} 
      ${params.statusCode ? `AND status_code=${params.statusCode}` : ''}
      LIMIT 1
    ) RETURNING *`
}
```
Using the same logic, you can parametrize your database bootstraps in any way you need.

