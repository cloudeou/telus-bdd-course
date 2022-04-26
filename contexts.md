### BDD framework training. Using Contexts

[![N|Solid](https://images.ctfassets.net/fikanzmkdlqn/5NoHRB1q6lrNzSSpekhrG5/cf22f3d7d9e82aed5e79659800458b57/TELUS_TAGLINE_HORIZONTAL_EN.svg)](https://www.telus.com/en/)

##### [Framework documentation](https://github.com/telus/telus-bdd-docs)

In this section you will understand the usage of contexts for data saving.

<img src="https://cdn4.iconfinder.com/data/icons/48-bubbles/48/23.Videos-512.png" width="30px" margin-top="15px"/> [Session video recording]()

[<img src="https://cdn-icons-png.flaticon.com/512/25/25231.png" width="30px"> Code Example](https://github.com/telus/bdd-demo/tree/master/bdd/contexts)

#### Contents:

1. [Context definition](#context-definition)
2. [Indentificators file](#indentificators-file)
3. [Code requirements](#code-requirements)
4. [Class structure best practice (accessors, map only)](#class-structure-best-practice-accessors-map-only)
5. [featureContext util usage](#featurecontext-util-usage)

#### **Context definition**
Contexts are in-memory data storages during runtime, a simple data storage, that can be used to save data inside of one framework launch between steps, scenarios and features(if multiple features where launched in one run).

In terms of javascript, Contexts are classes that store data in their fields.
#### **Indentificators file**
The mandatory part of each Context is its unique identificator, a string value used to work with contexts in the framework, framework uses Identificators to load your contexts to its runtime, and you use identificators to load contexts from the frameworks runtime.

>So before creating any contexts, you would first need to create identificators for them.

Context identificators are managed in one file `Identificators.ts`, which is located in the same directory as contexts, by default it is `./bdd/contexts`, but you can allways change their location by chahging `contextsPath` field in a config [file](./framework-intro.md/#bddconfig-file). The file is automatically created while framework initialization and looks like this:
```typescript
export const Identificators = {
  PIContext: "PIContext",
};
```
Inside of Identificators object are key value pairs of identificators.
> Note: identificator key and value must be identical
#### **Code requirements**
Now we can create a context, 
1. In the same directory, create a file with the same name as Identificator. 
In our example `PIContext.ts`.
2. Inside of this file import `Identificators.ts` and declare an empty class.
```typescript
import { Identificators } from "./Identificators";

export default class PIContext {}
```
3. Create public `identificator` property in the class and bind the corresponding identificator from inported file.
```typescript
import { Identificators } from "./Identificators";

export default class PIContext {
    public identificator = Identificators.PIContext;
}
```
Other code and design is up to you, you can implement your contexts as you would like, but we recomment to use best practice.
#### **Class structure best practice (accessors, map only)**
As a best practice we recommend using two approaches:
1. Getter/setter approach requires a considerable amount of code in context class, but creates convenient interface for development and support, where you would get hints on context data points names and data types, this will also help to reduce errors before execution.
In this approach each context data point is implemented with public field or private field and getter and setter methods if you want to add some additional logic(e.g. validation) for the data point or if you'd kust like to do it like this.
```typescript
export default class PIContext {
  public identificator = Identificators.PIContext;
  private _ecid: string = "";
  public response: { [key: string]: any } = {}; // public data point
  private _status_code: number = 0;

  // just plain getter/setter  
  public get ecid(): string {
    return this._ecid;
  }
  public set ecid(id: string) {
    this._ecid = id;
  }

  // getter/setter with additional validation logic
  public get status_code(): number {
    return this._status_code;
  }

  public set status_code(code: number) {
    if(code % 1000 != code) throw Error(`Not a valid http code: ${code}`)
    this._status_code = code;
  }
}
```
2. Map only approach, where the code inside of the context is minimal, but you would not receive any hits about context data during development.
In this approach you would only have one data point in the context, which would be of `Map` type, `Map` itself is capable of saving data points under some names, so you don't need to list all of your data points in code.
```typescript
export default class PIContext {
    public data: Map<any, any> = new Map();
}
```
#### **featureContext util usage**
To access contexts in step files you should use a special built in util `featureContext`. 

1. To use it you first need to import it from the framework, also inport identificators file, we would need it to access the context.
```typescript
const { featureContext } = require("@telus-bdd/telus-bdd");
import { Identificators } from "../contexts/Identificators";
```
2. **INSIDE OF** steps container function, create a function that will call the utils `getContextById` method to fetch context from the framework y its id:
```typescript
export const PISteps: StepDefinitions = ({ given, and, when, then }) => {
  const PIContext = (): PIContext =>
    featureContext().getContextById(Identificators.PIContext);
    // ...
}
```
3. Call created function inside of step definition callbacks to access context data:
```typescript
and(/^set customer (.*): (.*)$/, (paramName, paramValue) => {
    console.log(`Setting ${paramName} to ${paramValue}`);
    switch (paramName) {
      case "ecid":
        return (PIContext().ecid = paramValue);
      case "lpdsid":
        return (PIContext().lpdsid = paramValue);
}
``` 