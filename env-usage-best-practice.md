### BDD framework training. Environment usage best practice

[![N|Solid](https://images.ctfassets.net/fikanzmkdlqn/5NoHRB1q6lrNzSSpekhrG5/cf22f3d7d9e82aed5e79659800458b57/TELUS_TAGLINE_HORIZONTAL_EN.svg)](https://www.telus.com/en/)

##### [Framework documentation](https://github.com/telus/telus-bdd-docs)

In this section you will understand why environments are used and learn best practices to implement environments usage for the framework.

<img src="https://cdn4.iconfinder.com/data/icons/48-bubbles/48/23.Videos-512.png" width="30px" margin-top="15px"/> [Session video recording]()

#### **Contents**:

1. [Environments usage profit](#environments-usage-profit)
2. [Framework CLI parameter for env](#framework-cli-parameter-for-env) 
3. [Creating standard env-config module](#creating-standard-env-config-module)
4. [.env files and npm script to use them](#dotenv-files-and-npm-script-to-use-them)


#### **Environments usage profit**

Let's look into a small example where we have an endpoint http://platform.com/receiver to send http request with basic authentication 
and 3 separate environments where we need to test this endpoint:
- it01 - http://platform-it01.com/receiver , user1:password1
- it02 - http://platform-it02.com/receiver , user2:password2
- it03 - http://platform-it03.com/receiver , user3:password3

One way to test all of these steps is to pass environment parameter to step as follows:
```gherkin
When send request to receiver 'it01'
```
And process this parameter with conditional logic as follows:
```typescript
when(/^send request to receiver (.*)$/, async (env) => {
    switch(env) {
        case 'it01': {
            await axios({
                url: 'http://platform-it01.com/receiver', 
                auth: {
                    username: 'user1', 
                    password: 'password1'
                }
            });
            break;
        },
         case 'it02': {
            await axios({
                url: 'http://platform-it02.com/receiver', 
                auth: {
                    username: 'user2', 
                    password: 'password2'
                }
            });
            break;
        },
         case 'it03': {
            await axios({
                url: 'http://platform-it03.com/receiver', 
                auth: {
                    username: 'user3', 
                    password: 'password3'
                }
            });
            break;
        }
    }
});
```
And this would quite work, but there are couple of problems: 
- feature file must be modified to change environment each time. or copies of this file should be created with different param values
- all of the steps would need to receive such parameter, or separate given step should be created, which will also result in unnecessary field in context.

There are other ways to solve this problem, some are better, some are worth, this is why we have created a native support for environemtns in our framework and developed an approach, tested in many projects, to process multiple environments.

#### **Framework CLI parameter for env**

In order to use environemnt in the framework, you can use cli parameter `--bddEnv` on launch with string value of environment name, like this:

`npm run bdd:start -- --bddEnv=it01`

Using the value of this parameter framework populates Node.js enviromental variable to process.env of the launch process. This way it can be accessed from any place in your code. 
Based on this we can create a standard module to manage env configs.
 
#### **Creating standard env-config module**

Let's create a standard js module, that will manage envs based on framefork process.env. 

> Creation of this module will be automated in future versions, but it's structure and configuration will remain the same.

1. First you need to create `env-config` folder inside root directory of the project. Inside of this directory you need to create a number of `.ts/js` files that correspond to your environments (they should be named the same as envs) and `index.ts/js` file.

```
|
└─── env-config
        index.ts
        it01.ts
        it02.ts
        it03.ts
```

2. Now let's add env data to each env config:

```typescript
// it01
export const envConfig = {
    url: 'http://platform-it01.com/receiver',
    user: 'user1',
    password: 'password1'
}
```
```typescript
// it02
export const envConfig = {
    url: 'http://platform-it02.com/receiver',
    user: 'user2',
    password: 'password2'
}
```
```typescript
// it03
export const envConfig = {
    url: 'http://platform-it03.com/receiver',
    user: 'user3',
    password: 'password3'
}
```

3. Index file would export a config which corresponds to framework environment as such:
```typescript
// index
const bddEnv = process.env.bddEnv || 'it01'; /* getting framework env and setting default env if it is nit provided */
export const { envConfig } = require(`./${bddEnv}`); /*export dynamic require*/
```

4. That's it, now you can simply use this module to access env data without any changes to your code.
```typescript
import {envConfig} from './env-config';

when(/^send request to receiver$/, async () => {
    await axios({
        url: envConfig.url, 
        auth: {
            username: envConfig.user, 
            password: envConfig.password
        }
    });       
});
```
It is obvious that code became much shorter and much more clear and `envConfig` values will always correspond to needed environment.

#### **dotenv files and npm script to use them**

Now that we have our environment management managed there is another problem. Data is hardcoded to env configs, it has two potential flaws:
- Sesnsitive data as passwords is not secured, it is visible in the file
- Some data may cfrequently change and be accessible only from inside of real environments where BDD automation would be deployed.

Those can be solved by, again, using environmetal variables from process.env. Passwords and other data is often populated to the environment as enviromental variables in process of deployment or build pipelines (e.g. in Docker, Kubernetes, etc.)
And this is why env-config files are `.ts/js` and not `.json` or `.yaml`, it is possible to use Node.js `process.env` module to access OS environmental variables.

So an env-config file wuold look like this:
```typescript
// it01
export const envConfig = {
    url: 'http://platform-it01.com/receiver',
    user: process.env.USER_IT1,
    password: process.env.PASSWORD_IT1
}
```
 Security and accessability problems are solved, the last question is how to set up development environment for such approach?
 You can use **.env** file and **env-cmd** cli util to populate environmental variables to your Node.js process (recreating a real env).

1. First, create `.env` file in your project root directory with key-value pairs of variables:
 ```env
 USER_IT1=user1
 PASSWORD_IT1=password1
 USER_IT2=user2
 PASSWORD_IT2=password2
 USER_IT3=user3
 PASSWORD_IT3=password3
 ```

2. Then install env-cmd:
```npm
npm install env-cmd
```

 3. Modify `bdd:start` npm scrit in `package.json`:

 ```json
"scripts": {
    "bdd:start": "env-cmd bdd",
  },
```
**env-cmd** would look for `.env` file in root and populate all its variables to `process.env`

4. Finally add `.env` file to `.gitignore` to avoid sensetive data leak.
```gitignore
# env data
.env
``` 

Now when you launch automation on your local it would have the same environment as in deployments.
