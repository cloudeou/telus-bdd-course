### BDD framework training. Installation and initialization

[![N|Solid](https://images.ctfassets.net/fikanzmkdlqn/5NoHRB1q6lrNzSSpekhrG5/cf22f3d7d9e82aed5e79659800458b57/TELUS_TAGLINE_HORIZONTAL_EN.svg)](https://www.telus.com/en/)

##### [Framework documentation](https://github.com/telus/telus-bdd-docs)

In this section you will discover how to set up your workspace with BDD framework.

<img src="https://cdn4.iconfinder.com/data/icons/48-bubbles/48/23.Videos-512.png" width="30px>[Session video recording](https://drive.google.com/file/d/1wM_Fi1bEGYqYrdeNhSxLSDurDqwFUZQg/view?usp=sharing)

#### Contents:

1. [Installation using npm](#installation-using-npm)
2. [Initialization through cli](#initialization-through-cli)
3. [Typescript configuration and build script](#typescript-configuration-and-build-script)
4. [Git initialization](#git-initialization)

#### Installation using npm

BDD framework is a Node.js based application. In order to install it you should have Node.js installed.
Framework is distributed as npm package through GitHub packages npm registry.
The registry is private so in order to access it you will need an access token, for this please contact:

- dlFIFATestAutomationDevelopers@telus.com
- mykhailo.kosiuk@telus.com
- artem.zhiliaiev@telus.com

Once you have the token, you can start setup process.

1. Create an empty folder for your bdd project e.g. `my-bdd-project`
2. Open terminal and cd to your directory or open built in IDE terminal
3. Run `npm init -y` command to initialize npm in your project
4. Create `.npmrc` file and put registry name and token for this registry, that you should have, inside as follows:
   ```npmrc
   //npm.pkg.github.com/:_authToken=YOUR_ACCESS_TOKEN
   @telus-bdd:registry=https://npm.pkg.github.com
   ```
5. Now you are ready to install the package, run `npm install @telus-bdd/telus-bdd`

After this steps you should have created:

- `node_modules` directory with framework and other dependencies
- `package.json` file with npm settings and scripts

#### Initialization through cli

Now as the framework is installed, you need to initialize it in your project.

1. Go to `package.json` file and create new npm script inside to initialize the framework as follows:

```json
"scripts": {
  "bdd:init": "bdd-init"
}
```

2. Run `npm run bdd:init` in terminal and follow the instructions, answer yes to all questions as we will go through all framework features later, you would have the following output in the terminal:

```
npm run bdd:init

> bdd-init

? Do you use TypeScript in your project? Yes   // we strongly recommend using typescript
? Do you want to use Database bootstrap? Yes
? Do you want to use different environments? Yes
? Do you want to use contexts? Yes
? Enter default number of parallel threads to run your tests on: 1
? Do you want to create testOrder config for dependent features? Yes
? Do you want to use uReport reporter? Yes
? Do you need to integrate your tests into Spinnaker pipeline? Yes
```

We strongly recommend using typescript in your project as it provides static tipization, which will allow to write more clear and supportable code.
Framework is now initialized in your project.

#### Typescript configuration and build script

Now that the framework is initialized, we will initialize typescript (this procedure will be automated in future versions)

1. If you don't have typescript installed globally on your system, run `npm install typescript`
2. Now for initialization run `npx tsc --init`
3. You should have `tsconfig.json` file created in your project, go to this file, and replace its contents for the following:

```json
{
  "compilerOptions": {
    "module": "commonjs",
    "moduleResolution": "node",
    "sourceMap": false,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "preserveConstEnums": true,
    "removeComments": false,
    "noImplicitAny": false,
    "target": "ES2020",
    "outDir": "dist",
    "resolveJsonModule": true /* Disallow inconsistently-cased references to the same file. */,
    "rootDir": "./" /* Specify the root directory of input files. Use to control the output directory structure with --outDir. */,
    "esModuleInterop": true /* Enables emit interoperability between CommonJS and ES Modules via creation of namespace objects for all imports. Implies 'allowSyntheticDefaultImports'. */,
    "skipLibCheck": true /* Skip type checking of declaration files. */,
    "forceConsistentCasingInFileNames": true
  },
  "exclude": [
    ".eslintrc",
    ".gitignore",
    ".npmrc",
    "package-lock.json",
    "tsconfig.json"
  ]
}
```

4. Go to `package.json` file and create new script for build:

```json
"scripts": {
    "bdd:init": "bdd-init",
    "build": "npx rimraf dist && tsc",
  },
```

5. Now you can run `npm run build` and the project would be build into optimized raw javascript version (`dist` folder would be created after running the script, it contains the built code)
6. Now we need to slightly modify framework configuration to use built code from `dist` directory, for this go to `bdd.config.json(js)` file and put the following values inside:

```json
"stepsPath": "./dist/bdd/steps",
"contextsPath": "./dist/bdd/contexts",
```

#### Git initialization

The last thing to do is initialize git for your project:

1. Run `git init`
2. Create `.gitignore` file and put the following contents inside:

```gitignore
# Dependency directories
node_modules/
# node-waf configuration
package-lock.json
# build directory
dist/
```

**Congrats, you have finished project setup with npm, BDD Framework, Typescript and git**
