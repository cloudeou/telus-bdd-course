### BDD framework training. CLI Usage

[![N|Solid](https://images.ctfassets.net/fikanzmkdlqn/5NoHRB1q6lrNzSSpekhrG5/cf22f3d7d9e82aed5e79659800458b57/TELUS_TAGLINE_HORIZONTAL_EN.svg)](https://www.telus.com/en/)

##### [Framework documentation](https://github.com/telus/telus-bdd-docs)

In this section you will learn how to use framework CLI and write npm scripts to use it.

<img src="https://cdn4.iconfinder.com/data/icons/48-bubbles/48/23.Videos-512.png" width="30px" margin-top="15px"/> [Session video recording]()
#### Contents:

1. [Available npm package binaries](#available-npm-package-binaries)
2. [Best practice for npm scripts creation](#best-practice-for-npm-scripts-creation)
3. [Launch arguments](#launch-arguments)

#### **Available npm package binaries**
Npm package binaries are executable files that are added to OS path variables and can be launched from terminal or command line. These files usually are launchers or contain scripts for some installation or customization processes. 

BDD framework provides 2 binaries:
- `bdd-init` - framework project initializer that helps to create all default folders and config files.
- `bdd` - framework launcher.

It is important, that usage of these binaries is different for different operating systems:

On Linux distributives and MacOS most frequently binaries will be available to use in terminal juat as is by their name.

On Windows and for cases when binaries are not recognized by OS, we recommend using `npx` package runner, it automatically recognizes a needed environment and runs a command on any system.

So if the binaries are not recognized in tour terminal by their names as is:
1. Make sure you have `npx` installed by running `npx`
2. If the command is not recognized, run `npm install npx`
3. Then you will be able to run e.g. `npx bdd-init`
#### **Best practice for npm scripts creation**
Previous approach is good if you don't need to run binaries frequently, which is mostly the case for `bdd-init`, but you would surely frequently run `bdd` to launch your automations. 

For such cases it is better to create an npm script (though you can also create an npm script for `bdd-init`). We recommend to use `:` as separator in npm script names.

We recommend to have 4 npm scripts for every BDD project:
```javascript
"scripts": {
    "build": "tsc", // build
    "bdd:init": "bdd-init", // project initializer
    "bdd:start": "bdd", // launcher
    "bdd:start:dev": "env-cmd bdd" // uses .env file to populate env variables in dev env
  },
```

See [how to use `.env`](/env-usage-best-practice.md/#dotenv-files-and-npm-script-to-use-them)

#### **Launch arguments**
So, to launch your automation in bdd framework you need to run `npm run bdd:start` script, to pass any parameters to it, you need to prepend them with `--` empty parameter, that is the rule for launcing with npm script, e.g.:

`npm run bdd:start -- --suiteName=PI`

You can pass many arguments to this command:

- location to framework files and directories as in `bdd.config` (argument values would be prioritized over config file values)
- database bootstrap related arguments
- dependent features and uReport arguments

Full list can be seen by running `bdd --help` or `npx bdd --help`:

```
npx bdd --help

Options:
      --featuresPath   Path to features folder                          [string]
      --stepsPath      Path to steps folder. Specify path to dist/* for
                       TypeScript projects.                             [string]
      --envsPath
      --contextsPath   Path to contexts folder. Specify path to dist/* for
                       TypeScript projects.                             [string]
      --threadsNumber  Number of processes to run in parallel           [number]
      --testOrderPath  Path to tests order config file                  [string]
      --bootstrapPath  Path to custom bootstrap config file             [string]
      --config         Path to non-default config file                  [string]
      --suiteName      Identification tag of features to launch.        [string]
      --bddEnv         Framework launch environment name
      --runTimes       Customer number of run times                     [number]
      --useUReport     Should the framework generate and send reports to uReport
                       Platform.(Specify config in bdd.config for that)[boolean]
      --dataset        Id of dataset to use                             [number]
  -h, --help           Show help                                       [boolean]
```


