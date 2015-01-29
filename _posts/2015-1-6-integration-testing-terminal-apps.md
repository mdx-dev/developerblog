---
layout: post
author: blainesch
title: "Integration testing Command Line Apps"
description: "Test your node command line apps using cucumber with ease."
tags: [javascript, node, cucumber, cucumberjs, testing]
---

You might have seen [Aruba by Cucumber](https://github.com/cucumber/aruba), which helps you test your command line apps. For this article, we are going to be using [cucumber.js](https://github.com/cucumber/cucumber-js).

I decided to whip up a new command line app that has some interaction. It has three commands:
* add {ingredient}
* remove {ingredient}
* list

## Running your app

First things first we need to create the scenario that actually runs your script. In your [Cucumber World](https://github.com/cucumber/cucumber-js#world) you need to spawn a new process and save all the output.

~~~ coffeescript
# ./features/step_definitions/terminalStepDefinitions.coffee
@Given 'I open the shopping list', (callback) ->
  @runCommand 'coffee src/index.coffee', callback
~~~

~~~ coffeescript
# ./features/support/world.coffee
class World

  process: null

  output: []

  runCommand: (command, callback) =>
    command = command.split(' ') # the first command, in this case "coffee"
    @waitForOutput(callback)
    @process = spawn(
      command[0], # the command "coffee"
      command.slice(1), # rest of the arguments: ['src/index.coffee']
      cwd: "#{__dirname}/../../" # execute this in the root of the project
    )
    @process.stdout.on 'data', (data) =>
      @output.push data.toString() # store raw data
~~~

You'll notice the `waitForOutput`- we'll get to that in a bit.

## Input and Output

Here, all you need to do is write to the `STDIN` of the process. This also uses the magical `waitForOutput` command.

~~~ coffeescript
type: (input, callback) =>
  @waitForOutput(callback)
  @process.stdin.write "#{input}\n"
~~~

Depending on the type of program you are testing, you will need to modify the way you get output. For example if your program asks for input, part of the output is your question or prompt. Other programs don't have interaction, just output.

~~~ coffeescript
# Subtract 2
# 1 for index to count
# 1 account for the new prompt it sends
getOutput: (callback) =>
  callback(@output[@output.length - 2])
~~~

## Waiting for output

Some processes take a long time; for example, when you hit a database or a third party service. So waiting for output is necessary to get correct output. However, this isn't as magical as I've made it out to be. We see how much output we currently have, and keep checking until it increases by at least one.

The first step is to see how many we currently have and pass that into the real function. Additionally, you may pass in a date object if you want this to timeout after a given amount of time.

~~~ coffeescript
waitForOutput: (callback) =>
  @_waitForOutput(@output.length, callback)
~~~

Second, we check to see if the new output is bigger, otherwise we use `setTimeout` to call ourself again.

~~~ coffeescript
_waitForOutput: (oldOutputLength, callback) =>
  if @output.length > oldOutputLength
    callback()
  else
    setTimeout(@_waitForOutput.bind(@, oldOutputLength, callback), 500)
~~~

## Conclusion

Given this new world class you can make easy-to-use step definitions specific to your application that runs programs, types, and gets output. This allows you to test virtually any command line interface.

I created a repo to demonstrate this process and it's available on [Github](https://github.com/blainesch/shopping-list).
