---
title: Parallel Appium Test With Robot Framework (Pabot)
date: 2021-04-15
modified: 2021-04-15
categories: [appium, automated test]
excerpt: Make your Robot Framework Appium tests run faster through parallel execution!
tags: [appium, automated test]
classes: wide
header:
  overlay_image: /assets/images/robot-framework-parallel/header.jpg
  overlay_filter: 0.7 # same as adding an opacity of 0.5 to a black background
  caption: "Photo by [Azamat E](https://unsplash.com/@picsaf) on [Unsplash](https://unsplash.com)"
---

## [Robot Framework](https://robotframework.org/){:target="_blank"}

Robot Framework is a generic open source automation framework. It can be used for test automation and robotic process automation (RPA). It has easy syntax, utilizing human-readable keywords. The framework has a rich ecosystem around it, consisting of libraries and tools that are developed as separate projects.

In my opinion it is a straightforward and powerful tool that enables us to do testing. You can do almost all kinds of testing imaginable with Robot Framework. Included in one of them is mobile app automated testing using Appium.

Any organization wanting to kick-start automated testing process quickly with low learning curve should in my opinion give the framework a try. It can even implement tests in a `Given-When-Then` or Gherkin style (This sample project utilizes such approach).

Example:

- Test Scenario:
```
# Test Cases
Successful Division Operation
    Given User open calculator app
    When User perform division
    Then User see correct result of division
```

- Test Steps:
```
User perform addition
    Tap number 8
    Tap addition
    Tap number 8
    Tap equals
```

- Page-based Action:
```
Tap number 8
    Wait Until Element Is Visible   ${button.8} 
    Click Element                   ${button.8}
```

## Robot Framework Appium Test Structure
_Notes: The sample provided is to run tests on Android. To make it run on iPhone, you will need to tweak it here and there._

# [Repository](https://github.com/dnomyar90/RF-Appium-Parallel-Sample){:target="_blank"}

The structure of this sample project is as following:
```
- asset
    - calculator.apk:   Sample APK file to run tests.

- src
    - scenario:         Test cases on the upper-most level
    - step:             Page-based actions summarizing test case in the scenario level
    - page:             Individual page-based action
                        E.g. clicking, inputting text, checking text, etc.

- DevicesSet.dat:       Data devices to run the tests into (UDID, Appium Port, System Port)

- TestSetup.robot:      Test Setup and Test Teardown to run before and after each test

- run-test.sh:          Runner to simplify commands of running tests.
                        Two params: Tag of test to run and number of devices to run into.
```


The magic of the parallelization actually lies on using [pabot](https://pabot.org/){:target="_blank"} and resource file to distribute testing task to devices. We are going to run `Setup Test` before executing each test case. On `Setup Test` we get value set of devices we set. Below is the resource file sample:

```
[Redmi Note 9]
tags=redmi
udid=4aba4aacac3f71ce
appium_port=4723
system_port=8202

[Samsung S21 Ultra]
tags=samsung
udid=9RGNU20920100424
appium_port=4724
system_port=8201
```

We then capture the data set by utilising pabot's powerful `pabotLib` in order to ensure that only one of the processes uses the shared data set (Avoiding race condition). We then use the mutually exclusive acquired data to open application on the designated device.

```
Setup Test
    ${valuesetname}=    Acquire Value Set
    Log to console    ${valuesetname}
    ${used_udid}=     Get Value From Set   udid
    Set Global Variable    ${used_udid}
    Log to console    ${used_udid}

    ${used_appiumPort}=     Get Value From Set   appium_port
    Set Global Variable    ${used_appiumPort}
    Log to console    ${used_appiumPort}

    ${used_system_port}=     Get Value From Set   system_port
    Set Global Variable    ${used_system_port}
    Log to console    ${used_system_port}

    ${package}=    Set Variable        com.google.android.calculator
    ${appium_server}    Set Variable    http://127.0.0.1:${used_appiumPort}/wd/hub

    Open Application
    ...   ${appium_server}
    ...   platformName=Android
    ...   appPackage=${package}
    ...   appActivity=com.android.calculator2.Calculator
    ...   noReset=false
    ...   udid=${used_udid}
    ...   systemPort=${used_system_port}
    ...   automationName=uiautomator2
    ...   newCommandTimeout=10000
```

After the test execution is completed, it will run the `Test Teardown` defined on `TestSetup.robot`. Basically it will release the resources it was using and then closes the application currently undergoing test. Then the result will be presented to you as either failed or passed.

```
Teardown Test
    Release value set
    Close Application
```

In this sample, you will need to manually:
- Install APK to devices you want to run the tests with
- Start Appium server on ports you are going to use

Of course you can improvise and make the process automatic (We do it like that in our [company](https://kumparan.com/){:target="_blank"}. This sample project is intended to give the bare-minimum and easy to follow process of how to achieve parallel testing on Robot Framework with Appium.

Please kindly follow the pre-requisite of installing neccessary libraries and how to run the tests on the [repository](https://github.com/dnomyar90/RF-Appium-Parallel-Sample){:target="_blank"}

## Demo
Demo of parallel execution: 3 test cases run parallelly on 2 devices.
Test cases are done toward calculator APK sample:
- _Addition_
- _Division_
- _Multiply_

[![Parallel RF Appium Tests](https://j.gifs.com/jZ74m4.gif)](https://www.youtube.com/watch?v=u0nHLsjJnqc)asset

Testing tasks are distributed between available devices and avoiding race condition.

## Conclusion
This post is intended to show and encourage people who may feel reluctant to use Robot Framework for automated app testing. Some people I know express their concern about the lack of support and available documentation to run Appium tests paralelly on Robot Framework.

It is a valid concern for me though, since I did not find any material or sample projects that demonstrate the ability of Robot Framework to run Appium tests paralelly. I consider myself an expert in terms of googling things. So it is naturally shocking for me to see such lack of material on the internet.

Hopefully this post gives you an insight that it is possible to run Appium tests paralelly for tests written in Robot Framework.

_In our [company](https://kumparan.com/){:target="_blank"} we even go one step further by not using hardcoded value set. We are using endpoints and storing device state dynamically on the database for the sake of our internal QA services._

That's it for now. If you have any question or suggestion please kindly left a comment on the comment box below (_You'll need to use your Facebook account 😄✌️_)

_Stay safe, stay healthy, and always wear your masks outside!_

# [Repository](https://github.com/dnomyar90/RF-Appium-Parallel-Sample){:target="_blank"}