# Project description

Dynamics integration is a set of tools built on top of node.js that allows you to quickly create scripts to move data between Microsoft Dynamics CRM and other systems

# Requirements
TBC

Requires [Node.js](nodejs.org)

This tool is based on [Edge](https://github.com/tjanczuk/edge) and requires .NET Framework 4.5 to be installed.

# How to use it
## Install
Create a new directory to store your scripts

```
$ mkdir MyScripts
```

Then cd into it
```
$ cd MyScripts
```

Install Dynamics Inegration
```
$  npm install dynamicsintegration
```
## Start writing code

Create a new js file named myscript.js and start adding code

``` javascript

var CRMClient = require("DynamicsIntegration").CRMClient;

var crm = new CRMClient("Url=http://crm.contoso.com/xrmContoso");

var who = crm.whoAmI();
var myUser = crm.retrieve("systemuser",who);
console.log(myUser);

var mybu = crm.retrieve("businessunit",{businessunitid:myUser.businessunitid});
console.log(mybu);
```

Replace the connection string with your CRM server values.

More information about the connection string options: https://msdn.microsoft.com/en-us/library/gg695810.aspx

Save the file and run the script.

In order to run the script use node
```
$ node myscript
```
# CRMClient methods
## Constructor
``` ts
constructor(private connectionString?: string)
```
The connection string could be either a valid connection string or a name of an existing connection string in the file "config.json" at the root path.

The config.json file must be a valid json file, with a property named connectionStrings, with the following format.

``` json
{
	"connectionStrings":
	{
    "default":"Url=http://crm.contoso.com/xrmContoso; Domain=CONTOSO; Username=jsmith; Password=passcode",
    "connection2":"Url=http://crm.contoso.com/xrmContoso"
	}
}
```
If no value is passed to the constructor, the "default" text will be assumed, which means that a connection string named "default" will be used.

Example:
``` javascript
var crm = new CRMClient("Url=http://crm.contoso.com/xrmContoso; Domain=CONTOSO; Username=jsmith; Password=passcode");
```
Creates a connection using a valid Connection String

``` javascript
var crm = new CRMClient("connection2");
```
Creates a connection using the connection string named "connection2" specified in the config.json file

``` javascript
var crm = new CRMClient();
```
or

``` javascript
var crm = new CRMClient("default");
```
Creates a connection using the connection string named "default" specified in the config.json file


More information about the connection string options: https://msdn.microsoft.com/en-us/library/gg695810.aspx

## WhoAmI

Returns the GUID of the current user. Useful for testing the active connection.

Example:
``` javascript
var who = crm.whoAmI();
console.log(who);
```
outputs '6fefeb79-5447-e511-a5db-0050568a69e2'

## Retrieve
Retrieves a record given its Guid

Signature:
``` typescript
retrieve(entityName: string, id: string|Guid, columns?: string|string[]|boolean)
```
**entityName** is the name of the entity to be retrieved. The name is case insensitive, so all values are lowercased before send to CRM.

**id** can be either a string with the GUID if the record to be retrieved, or a Guid object with the same value.

**columns** can be either a column name, an array of column names, or a *true* value indicating that all columns must be retrieved. The default value is *true*. An ***** value has the same effect

Examples:
``` javascript
var account = crm.retrieve("account","6fefeb79-5447-e511-a5db-0050568a69e2");
console.log(account)
```
Returns all the columns for the specified account id. The last is equivalent to
``` javascript
var account = crm.retrieve("account","6fefeb79-5447-e511-a5db-0050568a69e2","*");
console.log(account)
```
or to
``` javascript
var account = crm.retrieve("account","6fefeb79-5447-e511-a5db-0050568a69e2",true);
console.log(account)
```
Also can use the Guid class to specify the *id* parameter. This allows to perform a GUID validation before calling the method.
``` javascript
var Guid = require("DynamicsIntegration").Guid;
var account = crm.retrieve("account",new Guid("6fefeb79-5447-e511-a5db-0050568a69e2"));
console.log(account)
```

``` javascript
var account = crm.retrieve("account","6fefeb79-5447-e511-a5db-0050568a69e2",["accountid","name","ownerid","createdon"]);
console.log(account)
```
Returns the *accountid,name,ownerid,createdon* columns for the given account id

``` javascript
var account = crm.retrieve("account","6fefeb79-5447-e511-a5db-0050568a69e2","name");
console.log(account.name)
```
Returns the *name* of the specified account.

The returned value is a javascript object containing the values of the record. If no data found, then a *null* object is returned.

The returned object will contain also the name and type for *Optionset* and *Lookup* fields.

``` javascript
var account = crm.retrieve("account","6fefeb79-5447-e511-a5db-0050568a69e2","ownerid");
console.log(account.ownerid); // outputs the GUID value
console.log(account.ownerid_type); // outputs systemuser
console.log(account.ownerid_name); // outputs John Doe
```
## Retrieve multiple
Retrieves one or more records of the specified entity that fulfill the specified conditions.

Signature:
``` typescript
retrieveMultiple(fetchXml: string): Array<any>;
retrieveMultiple(entityName: string, conditions?, attributes?:boolean|string|string[]): Array<any>
```
**fetchXml** is a *Fetch Xml* query to specify the records to be retrieved. More information in https://msdn.microsoft.com/en-us/library/gg328332.aspx

**entityName** in case you don't want to write an xml to specify the records to retrieve, can use second overload for this method. The only mandatory parameter for this overload is the *entityName* that specifies the entity to be retrieved. If the attributes parameter is missing, the all the existing records of the specified entity will be retrieved. Is equivalent to write a Fetch Xml without any filter conditions.

**conditions** is a Javascript object that must contain a property for every filter to be applied. More info in [the next chapter](#conditions-object).

**attributes** Same as in the retrieve method. Specifies the columns to be retrieved. Could be either a single column name or an array of columns names or a *true* value to indicate the all columns have to be retrieved. The last is the default value.

Returns an array ob objects, where each item contains the values of one record.

Examples:
``` javascript
var accounts = crm.retrieveMultiple("<fetch><entity name='account'></entity></fetch>");
```
Retrieves all the records of the account entity. Only the accountid column will be retrieved (the id column is always returned in all Crm queries)
``` javascript
var accounts = crm.retrieveMultiple("account");
```
Retrieves all the records of the **account** entity. Includes also ***all*** the columns of the entity.
``` javascript
var accounts = crm.retrieveMultiple("account",{name:"contoso"});
```
Retrieves all the records of the **account** entity where the account name is equals to "contoso". Returns ***all*** the columns of the entity.
``` javascript
var accounts = crm.retrieveMultiple("account",{name:"contoso"},["accountid","name","ownerid","createdon"]);
```
Retrieves the same as the previous one, but only the specified columns are included in the query.
``` javascript
var accounts = crm.retrieveMultiple("account",{name:["contoso","acme"]});
```
Retrieves all the records of the **account** entity where the account name is equals to "contoso" or "acme". Returns ***all*** the columns of the entity.

As in the retrieve method, each returned object will contain the name and type for *Optionset* and *Lookup* fields.

## Conditions object
There are two ways to specify when retrieving records from CRM. A Fetch Xml query, or a condition object.

A condition object is a Javascript where every property represents a condition.

For example:

``` javascript
var cond = {name:"myAccount"}
```
means: retrieve all records where the attribute *name* is equals to *"myAccount"*

You can also add more conditions
``` javascript
var cond = {name:"myAccount", createdon:new Date()}
```
means: retrieve all records where the attribute *name* is equals to *"myAccount"* and the attribute *createdon* is equal to the current date

If you want to specify other operators, use the following Syntax
``` javascript
var cond = {name:{$like:"contoso%"}, createdon:{$bt:new Date()}}
```
means: retrieve all records where the attribute *name* begins with *"contoso%"* and the attribute *createdon* is bigger than the current date

The currently supported operators are:
```
$eq, $neq, $gt, $ge,	$le, $lt, $like, $notlLike, $in, $notIn, $between, $notBetween
```
when the simple syntax is used ``` {name:value} ``` then the ```$eq``` operator is used.

If the value is an array, then the ```$in``` operator is used. For example: ``` {name:["contoso", "acme", "cycleworks"]} ``` means: where name is one of contoso, acme or cycleworks.

If the value is a null value, then the ```null```operator is applied. For example: ```{name:null}``` will retrieve all the records where the name is ```null```.

## Retrieve all
Is a simplified way of retrieving all the existing records of an entity.

Signature:
```typescript
retrieveAll(entityName: string): Array<any>
```
Is equivalent to call the retrieveMultiple method not specifying the conditions or attributes method


## Create
Creates a record with the specified values in CRM and returns the Guid of the record.

Signature:
``` typescript
create(entityName: string, attributes: any): string
```
**entityName** the name of the entity which record is being created

**attributes** javascript object with the values to apply to the new record.

Examples:
``` javascript
var accountid = crm.create("account",{name:"contoso",description:"this is a test",AccountCategoryCode:1});
console.log(accountid);
```
As always, the names in the entity or attributes are case insensitive, so all the names will be lowercased before send the operation to Crm.


## Update
Updates one or more records that meet the specified conditions and returns the number of updated records.
Signature:
``` typescript
update(entityName: string, attributes: any, conditions?): number
```
Parameters:

**entityName** the name of the entity to be updated

**attributes** Javascript object with the values to apply to the records. The same as in the Create method.

**conditions** [conditions object](#conditions-object) that must contain a property for every filter to be applied.

Examples:
``` javascript
var affectedRecords = crm.update("account",{name:"contoso-updated"},{name:"contoso"})
```
Updates all the accounts which name is contoso, and set the attribute value to contoso-updated.

``` javascript
var affectedRecords = crm.update("account",{accountid:"6fefeb79-5447-e511-a5db-0050568a69e2",name:"contoso-updated"})
```
If you don't specify the conditions parameter, then you have to know the GUID if the record, and set it in the second parameter.

In this example, only the account with the specified account id will be updated. If the specified record id exists, then affectedRecords should be equals to 1.

## Delete
Deletes one on more records in CRM, and returns the number of records affected.

Signature:
``` typescript
delete(entityName: string, idsOrConditions):number
```
Parameters:

**entityName** the name of the entity to be updated.

**idsOrConditions** can be either a Guid, a string, an array or a conditions object. If it is Guid will delete the record with the specified id. If it is a string, must be a GUID, and again, will delete the records matching the specified id. If the parameter is an array, its elements must be either a string or a Guid, and in each case, the records deleted will be the ones specified by those GUIDS. If it is a condition object, firs, all the matching records will be retrieved, and then deleted.

Examples:
``` javascript
var affectedRecords = crm.delete("account","6fefeb79-5447-e511-a5db-0050568a69e2");
```
This will delete the account with the specified GUID.

``` javascript
var affectedRecords = crm.delete("account",new Guid("6fefeb79-5447-e511-a5db-0050568a69e2"));
```
The same as the previous example.

``` javascript
var affectedRecords = crm.delete("account",["6fefeb79-5447-e511-a5db-0050568a69e2","6fefeb79-5447-e511-a5db-0050568a69e2");
```
This will delete the two accounts with the specified ids.

``` javascript
var affectedRecords = crm.delete("account",{name:"contoso"});
```
This will delete all the accounts named contoso.

TBC

## Upsert
Checks if the specified record exists, and if it does, then it updates, otherwise, it creates a new one with the specified values.

TBC


# Repl integration
TBC

# Commands
TBC

## Export
```
$ di export systemuser users.xml
```

## Import
```
$ di import users.xml
```

# Supported CRM versions
Tested in CRM 2013 and CRM Online so far.

Volunteers to run the integration tests in other versions are welcome :)


# Use it as an integration tests tool
TBC

# How to compile the code
TBC

# How to run tests
Requires mocha to run the tests (https://mochajs.org/)
```
$  npm test
```
## Integration Tests
In order to run the integration tests, first download the code as described in before.

Then, you need create a config.json file in the root folder of DynamicsIntegration folder.

The config.json must have the following format:
```json
{
	"connectionStrings":
	{
		"IntegrationTests2015":"Url=https://contoso.crm4.dynamics.com; Username=admin@contoso.onmicrosoft.com; Password=YourPassword;"
	}
}

```
You need to create a connection string named "IntegrationTestsXXXX" where XXXX is the CRM version you want to test against. Can be 2011, 2013 or 2015.

That XXXX indicates what version of the CRM dlls do you want to use in order to run the tests.

You can add up to three different integration tests, oner per supported version, 2011, 2013 or 2015.

After doing that, just re run the tests with mocha.
```
$  npm test
```

You should see something like this:
``` console
> dynamicsintegration@1.0.0 test
> mocha --recursive



  CRMClient
    V Throws an exception with an invalid connection (249ms)

  Fetch
    V Serializes a simple Fetch
    V Serializes a simple Fetch with conditions
    V Serializes a complex Fetch

  Guid
    V Creates an instance of non empty Guid
    V Creates an instance of empty Guid
    V Creates an instance from a string
    V Validates Guid format on creation
    V Generates new Guids
    V Compares two different Guids
    V Compares two equal Guids
    V Compares a Guid value with a string

  Integration Tests: 2011
    V Creates an account (1653ms)
    V Updates an account (805ms)
    V Updates an account using a criteria (931ms)
    V Knows Who I am (43ms)
    V Performs a simple retrieve (114ms)
    V Performs a retrieve with specific columns (76ms)
    V Performs a retrieve with all columns (78ms)
    V Performs a "retrieve all" of an entity (97ms)
    V Performs a simple retrieve multiple (103ms)

  Integration Tests: 2015
    V Creates an account (2041ms)
    V Updates an account (884ms)
    V Updates an account using a criteria (933ms)
    V Knows Who I am (88ms)
    V Performs a simple retrieve (318ms)
    V Performs a retrieve with specific columns (180ms)
    V Performs a retrieve with all columns (177ms)
    V Performs a "retrieve all" of an entity (221ms)
    V Performs a simple retrieve multiple (264ms)


  30 passing (9s)
```

# Backlog
(Ordered by priority)

* *Improve documentation (Work In Progress)*
* Upload npm package
* Add DataTable functions
* ~~Add different Crm versions support~~
* ~~Improve integration tests~~
* Add test coverage reports
* Add SQL support
* Add import/expport commands
* Add functions to work with solutions
* Add support for missing operators in condition objects
