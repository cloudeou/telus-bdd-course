### BDD framework training. Creating step files

[![N|Solid](https://images.ctfassets.net/fikanzmkdlqn/5NoHRB1q6lrNzSSpekhrG5/cf22f3d7d9e82aed5e79659800458b57/TELUS_TAGLINE_HORIZONTAL_EN.svg)](https://www.telus.com/en/)

##### [Framework documentation](https://github.com/telus/telus-bdd-docs)

In this section you will learn what are steps and how to create them, how to create reusable and parametrazable steps.

<img src="https://cdn4.iconfinder.com/data/icons/48-bubbles/48/23.Videos-512.png" width="30px" margin-top="15px"/> [Session video recording]()

[<img src="https://cdn-icons-png.flaticon.com/512/25/25231.png" width="30px"> Code Example](https://github.com/telus/bdd-demo/blob/master/bdd/steps/PI.steps.ts)

#### Contents:

1. [Steps (or action word) definition and structure](#steps-definition-and-structure)
2. [Steps file, creation best practice](#steps-file-creation-best-practice)
3. [Steps container and steps creation code requirements](#steps-container-and-steps-creation-code-requirements)
4. [String stepmatcher parametrization](#string-stepmatcher-parametrization)
5. [Regex stepmatcher parametrization](#regex-stepmatcher-parametrization)
6. [Table parameter](#table-parameter)
7. [Steps index file](#steps-index-file)


#### **Steps definition and structure**
Steps (aka action words) are the building blocks of scenarios and feture files. They are considered as the most low-level component of BDD. Hierarchy is:
- Features group (joined by tag)
    - Feature
        - Scenario
            - Step 

Each step represents a reusable piece of logic whether it is some action, condtition or precondition. Behind each step there is a real Node.js function that executes this logic based on argument values and context.

BDD step has a simple structure which includes:
- Keyword
- Step body (action word body)
- Parameters (in form of strings or tables)

    ```gherkin
        When     perform api call to url     'http://host.com/api/v1' with headers
    # keyword |          step body        |      string parameter 
        | header       | value            |
        | Content-type | application/json |  
        | Accept       | */*              |
     #           table parameters   
    ```
#### **Steps file, creation best practice**
Steps files are one of framework-specific implementation parts. By default all step files are located in `./bdd/steps` directory (and executed from `./dist/bdd/steps` if Typescript is used).

All step files should be located inside one parent directory as a requirement, but you can always change this directory in [`bdd.config` file](./framework-intro.md/#bddconfig-file) 

As a best practice naming of feature files should follow the next convention:

    camelCaseName.steps.ts[js]

In our example `PI.setps.ts`.
#### **Steps container and steps creation code requirements**
Inside of a steps file it is required to create a special container-function with the following syntax(from example):
```typescript
export const PISteps: StepDefinitions = ({ given, and, when, then }) => {
    // ...
}
```
So it's a function, which takes four keyword functions as argument and does not return anything (void). Inside of this function step definitions are implemented by calling keyword functions. And it is exported from the file.

All of the other code as creation of service class instances, contexts instances creation should be inside of a container-function before declaring steps (e.g.):
```typescript
export const PISteps: StepDefinitions = ({ given, and, when, then }) => {
  const fifaNcApi = new FIFANCApi();
  const PIContext = (): PIContext =>
    featureContext().getContextById(Identificators.PIContext);
    // ...
    //steps declaration
}
```

Steps are declared **ONLY** inside of a container-function as _step definitions_. Step definitions are function calls of keyword functions, which take two arguments:
- stepmatcher (`String` or `RegExp`)
- callback function (can be asynchronous)
```typescript
// with string step-matcher
when("call TMF Product Inventory API", async () => {
    // callback function code
});

// with regexp step-matcher
and(/^set customer (.*): (.*)$/, (paramName, paramValue) => {
    // callback function code    
});
```

> Note: 
> 
> Keyword function usage does not allways must be simillar to BDD step keyword to be used in feature files. For example:
> - `when()` step definition can be used with `When` or `Then` BDD keywords in a feature file
> - `and()` step definition can be used with all BDD keywords(`Given`, `When`, `Then`, `And`) in a feature file
> - all step definitions can be used with `And` BDD keyword in a feature file.
>
> e.g.
> ```typescript
> and(/^set customer (.*): (.*)$/, (paramName, paramValue) => {
>    // callback function code    
> }); 
> ```
>```gherkin
> Scenario: Set precondition data
>    Given set customer ecid: @ecid
>    And set customer lpdsid: @lpdsid
>```
> etc.
>

#### **String stepmatcher parametrization**
Step definitions can and should be parametrized. The more parameetrized the step is, the more reusable it may be. 

>Note:
>
> But there are cases when it is better to create separate step definitions instead of parametrizing one. For example when there is a big conditional logic depending on a parameter in callback function, it may theoretically slow down the code, because conditional code would run every time. 

String step-matcher can be parametrized in only one way, parameter value goes in the end of the step


For string parameter:

```typescript
and('response should be valid by schema ', (schemaName: string) => {
    /// ...
});
```
You should not forget about a space in the end to separate parameter value.
```gherkin
Scenario: Make Product inventory call
    And response should be valid by schema responseSchema.json
```
#### **Regex stepmatcher parametrization**
Regular expression step-matcher is much more flexible, you can put any number of parameters in any places in the string you would like as this:
```typescript
and(/^response should be valid by schema (.*)$/, (schemaName: string) => {
    // ...
});
```
or this:
```typescript
and(/^set customer (.*): (.*)$/, (paramName: string, paramValue: any) => {
    // ...
});
```
```gherkin
Scenario: Set migration data
    Given set customer ecid: @ecid
    And set customer lpdsid: @lpdsid
```

> Note:
>
> As a best practice we recommend to use only regex stepmatcher as it has much better abilities for this and it is easier to see parameters in code. And to use string step-matchers in cases when step does not require any parameters.
#### **Table parameter**
Except from string parameters step definitions can also be parametrized with table parameter. 
```typescript
and(/^response should be valid by schemas (.*):$/, (schemasTable: any) => {
    /// ...
});
```
As a best practice end table parametrized steps with ':', it would help to understand wether parameter is a string parameter or a table parameter.
```gherkin
Scenario: Make Product inventory call
    And response should be valid by schemas:
    | schema               | 
    | responseSchema.json  |
    | responseSchema1.json |
```
Table in terms of javascript is an array of objects, each object is a table row and has fields with keys corresponding to column names. So it can be processed in the following way in the code:
```typescript
and(/^response should be valid by schemas (.*):$/, (schemasTable: any) => {
    // creating array of shema names from table 
    const schemaNames: string[] = schemasTable.map((row) => {
        console.log(row);
        return row.schema;
    });
});
```
Tables can have any number of columns and table structure is very clear to read in a feature file, so if you have a complex parametres structure it is better to use table. 
#### **Steps index file**
Steps index file is located in the same folder as step files `./bdd/steps/index.ts`. 
It is main and only function is to control which step definitions are loaded to the framework. It is usefull when you temporarily do not need some steps, there is no sence to load them to the framework or for testing.
Steps index basically is a standard index file of a javascript module file and looks like this:
```typescript
import { PISteps } from "./PI.steps";

export default [PISteps];
```  
Particlularly by adding or removing step definitions from exported array you controlled whether they are or are not loaded to the framework.



