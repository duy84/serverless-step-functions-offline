[![serverless](http://public.serverless.com/badges/v3.svg)](http://www.serverless.com)
[![Maintainability](https://api.codeclimate.com/v1/badges/b321644ef368976aee12/maintainability)](https://codeclimate.com/github/duy84/serverless-step-functions-offline/maintainability)


[![NPM](https://nodei.co/npm/serverless-step-functions-emulator.png)](https://nodei.co/npm/serverless-step-functions-emulator/)

# serverless-step-functions-offline ![circleci status](https://circleci.com/gh/duy84/serverless-step-functions-offline.svg?style=shield)

## Documentation

- [Install](#install)
- [Setup](#setup)
- [Requirements](#requirements)
- [Usage](#usage)
- [Run plugin](#run-plugin)
- [What does plugin support](#what-does-plugin-support)
- [Usage with serverless-wepback](#usage-with-serverless-webpack)


# Install
Using NPM:
```bash
npm install serverless-step-functions-emulator --save-dev
```
or Yarn:
```bash
yarn add serverless-step-functions-emulator --dev
```

# Setup
Add the plugin to your `serverless.yml`:
```yaml
# serverless.yml

plugins:
  - serverless-step-functions-emulator
```

To verify that the plugin works, run this in your command line:
```bash
sls step-functions-offline
```

It should rise an error like that:

> Serverless plugin "serverless-step-functions-offline" initialization errored: Please add ENV_VARIABLES to section "custom"

# Requirements
This plugin works only with [serverless-step-functions](https://github.com/horike37/serverless-step-functions).

You must have this plugin installed and correctly specified statemachine definition using Amazon States Language.

Example of statemachine definition you can see [here](https://github.com/horike37/serverless-step-functions#setup).

# Usage
After all steps are done, need to add to section **custom** in serverless.yml the key **stepFunctionsOffline** with properties *stateName*: name of lambda function.

For example:

```yaml
service: ServerlessStepPlugin
frameworkVersion: "3"
plugins:
  - serverless-step-functions
  - serverless-step-functions-emulator

# ...

custom:
  stepFunctionsOffline:
    StepOne: firstLambda
    # ...
    StepTwo: secondLambda

functions:
  firstLambda:
    handler: firstLambda/index.handler
  secondLambda:
    handler: secondLambda/index.handler

stepFunctions:
  stateMachines:
    foo:
      definition:
        Comment: "An example of the Amazon States Language using wait states"
        StartAt: StepOne
        States:
          StepOne:
            Type: Task
            Resource: arn:aws:lambda:eu-west-1:123456789:function:${self:service}-${opt:stage, self:provider.stage}-firstLambda
            Next: StepTwo
          StepTwo:
            Type: Task
            Resource: arn:aws:lambda:eu-west-1:123456789:function:${self:service}-${opt:stage, self:provider.stage}-secondLambda
            End: true
```

Where:
- `StepOne` is the name of step in state machine
- Make sure `StepOne` has been defined in `custom.stepFunctionsOffline` and pointed to the Lambda function that has been defined in States. In this example is `firstLambda`
- `firstLambda` is the name of function in section **functions**

# Run Plugin
```bash
sls step-functions-offline --stateMachine={{name}} --event={{path to event file}}
```

- `name`: name of state machine in section state functions. In example above it's `foo`.
- `event`: input values for execution in JSON format (optional)

If you want to know where you are (in offline mode or not) you can use env variable `STEP_IS_OFFLINE`.

By default `process.env.STEP_IS_OFFLINE = true`.

# What does plugin support?
| States | Support |
| ------ | ------ |
| ***Task*** | Supports *Retry* but at this moment **does not support fields** *Catch*, *TimeoutSeconds*, *HeartbeatSeconds* |
| ***Choice*** | All comparison operators except: *And*, *Not*, *Or* |
| ***Wait***  | All following fields: *Seconds*, *SecondsPath*, *Timestamp*, *TimestampPath* |
| ***Parallel*** |  Only *Branches* |
| ***Map*** | Supports *Iterator* and the following fields: *ItemsPath*, *ResultsPath*, *Parameters* |
| ***Pass*** | Result, ResultPath |
| ***Fail***| Cause, Error|
| ***Succeed***| |

### Usage with serverless-webpack

The plugin integrates very well with [serverless-webpack](https://github.com/serverless-heaven/serverless-webpack).

Add the plugins `serverless-webpack` to your `serverless.yml` file and make sure that `serverless-webpack`
precedes `serverless-step-functions-offline` as the order is important:
```yaml
  plugins:
    ...
    - serverless-webpack
    ...
    - serverless-step-functions-emulator
    ...
```

### TODOs
 - [x] Support context object
 - [x] Improve performance
 - [x] Fixing bugs
 - [x] Support Pass, Fail, Succeed
 - [x] Integration with serverless-webpack
 - [x] Support Map
 - [x] Support field *Retry*
 - [ ] Support field *Catch*
 - [ ] Add unit tests - to make plugin stable (next step)
 - [ ] Support other languages except node.js
