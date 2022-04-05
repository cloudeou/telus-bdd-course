### BDD framework training. Creating a feature file

[![N|Solid](https://images.ctfassets.net/fikanzmkdlqn/5NoHRB1q6lrNzSSpekhrG5/cf22f3d7d9e82aed5e79659800458b57/TELUS_TAGLINE_HORIZONTAL_EN.svg)](https://www.telus.com/en/)

##### [Framework documentation](https://github.com/telus/telus-bdd-docs)

In this section you will learn what are feature files, how to properly and conviniently create them.

<img src="https://cdn4.iconfinder.com/data/icons/48-bubbles/48/23.Videos-512.png" width="30px" margin-top="15px"/> [Session video recording]()

#### Contents:

1. [Feature file definition](#feature-file-definition)
2. [Feature file structure](#feature-file-structure)
3. [Identification tags and their usage best practice](#identification-tags-and-their-usage-best-practice)
4. [Tag parameters](#tag-parameters)

#### **Feature file definition**

Feature file is 
- the most high-level file in BDD framework
- used as an input for the framework
- the place where you can modify and parametrize automation flow
- just a text file with `.feature` exntention, written in *Gherkin* language
- not an executable file but just a config file that can be consumed by BDD framework, which will execute described flow, that is also why changing features does not require any rebuilds or reboots of the deployment.

#### **Feature file structure**

```gherkin
@-               #
@-               # identification tags
@PIComponentTest #
@DBbootstrap=testbootstrap # tag params

Feature: Test new db bootstrap with dataset id from cli # feature title

  Scenario: Set migration data     # scenario title
    Given set customer ecid: @ecid   # action words
    And set customer lpdsid: @lpdsid #

  Scenario: Make Product inventory call
    Given developer selected ecid and lpdsid
    When call TMF Product Inventory API
    Then response status code should be @status_code
    And response should be valid by schema @schema
    And finish testing for @id
```

Feature file parts are: 
- *identification tags* 

    Tags that are used as identificators of feature files. They can be used in launch script to define which feature files need to be executed.
- *tag params*  

    Tags that are used as key-value parameters for a feature file, allowing to provide more configuration to a standard Gherkin functionality. With them you can specify database bootstrap, run times etc.

- *feature title*
    
    Title of the feature file, it is used in logs and results.
- *scenario title*

    Title of the scenario, it is used in logs and results.
- *action words*

    Action words are the building blocks of feature files, automation flows are described with them, action words correspond to real javascript functions that are executed in runtime.
    Action words start from *key words (Given, When, Then, And)*.
    Action words can be parametrized in two ways with a single parameter and with a table.

    ```gherkin
    # single parameter example, where 'migrated' is a parameter value
    And set flag: migrated
    ```
    ```gherkin
    # table parameters example
    And set flags:
    | FlagName  |
    | migrated  |
    | validated |
    ```

#### **Identification tags and their usage best practice**

Identification tags are a very important part of the feature file, they allow to specify which featurs need to be launched.

Identification has 3 levels, from the general, to the specific. You can think of these levels as:
- 1st level - class 
- 2nd level - sub-class
- 3rd level - element (must be unique)

For example in a BDD project you may have featurs used for **test automation**, among them are **ui tests** and **api tests**, and you have test cases **test1** and **test2**. Then your identification tags would look like this:
```gherkin
@test-automation
@ui-tests # or @api-tests
@test1 # or @test2
```
It is recommended as good practice to have `features` directory inner structure to correspond to your identificcation tags, for our example:
```
└───features
    └───test-automation
        ├───api-tests
        │       test2.feature
        │
        └───ui-tests
                test1.feature
```

Now when launching BDD framework you can use these tags to filter features to be executed, for example if you lanuch the framework with `--suiteName=test-automation` parameter all of the features would be executed for our example. 

#### **Tag parameters**

Tag parameters are also unique for BDD framework, they are used to provide additional meta data about the feature.
> **Syntax**: tag params are key-value pairs, '=' symbol is used to set parameter values.

Available tag parameters are: (see [database bootstrap](/database-bootstrap.md) for usage)
- `DBbootstrap` 
- `runTimes`
- `DBbootstrapParams`