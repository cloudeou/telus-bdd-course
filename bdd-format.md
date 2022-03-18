### BDD framework training. BDD and BDD format

[![N|Solid](https://images.ctfassets.net/fikanzmkdlqn/5NoHRB1q6lrNzSSpekhrG5/cf22f3d7d9e82aed5e79659800458b57/TELUS_TAGLINE_HORIZONTAL_EN.svg)](https://www.telus.com/en/)

##### [Framework documentation](https://github.com/telus/telus-bdd-docs)

In this section you will briefly learn what is BDD and discover BDD format used in the framework.

#### Contents:

1. [BDD format background](#bdd-format-background)
2. [BDD scenario and keywords](#bdd-scenario-and-keywords)
3. [BDD format in the framework](#bdd-format-in-the-framework)

#### BDD format background

Behavioral Driven Development (BDD) is a software development approach that has evolved from TDD (Test Driven Development). It differs by being written in a shared language, which improves communication between tech and non-tech teams and stakeholders. In both development approaches, tests are written ahead of the code, but in BDD, tests are more user-focused and based on the system’s behavior.

The advantage to BDD test cases being written in a common language is that details of how the application behaves can be easily understood by all. For example, test cases can be written using real-time examples of the actual requirements, to explain the behavior of the system.

#### BDD scenario and keywords

BDD is based on 4 keywords to build scenarios:

1. **Given** - preconditions(requirements) of scenario
2. **When** - scenario main action
3. **Then** - expected outcome of the action (or subsequent dependent action)
4. **And** - can be used to extend each of 3 main keywords

- BDD scenario is written using the keywords
- Each line should start from the keyword
- Scenario must contain **only one** _given_, _when_, _then_ keyword, thay can be extended using _and_ keyword
- You should try to stick to keyword meanings as a best practice (e.g. do not perform actions in _given_), this would help to create more consice scenarios.

#### BDD format in the framework

In the framework we support the same BDD logic, the only addition is `Scenario:` title used to give a name for a scenario.
BDD scenario in the framework looks like this:

```gherkin
Scenario: Some scenario
  Given some precondition
  And another precondition
  When perform some action
  Then expect something
  And expect something else
```
