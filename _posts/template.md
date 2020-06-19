---
title: JavaScript Selenium Mocha QuickStart Guide
date: 2016-07-20
modified: 2018-11-15
categories: [Selenium]
excerpt: A quick how-to on setting up a NodeJS project using Mocha and Selenium!
tags: [Selenium, Mocha, JavaScript, NodeJS, windows]
classes: wide
header:
  overlay_image: /assets/images/js-selenium-mocha/header.jpg
  overlay_filter: 0.5 # same as adding an opacity of 0.5 to a black background
  caption: "Photo by [Ilya Pavlov](https://unsplash.com/@ilyapavlov) on [Unsplash](https://unsplash.com)"
---

## Getting started with JavaScript, Selenium and Mocha!
In this article we'll be using Mocha and Selenium to write tests for web applications. This will be a starter project we can build on for various projects and in future articles.

### Requirements
Here are the requirements before we get started:

* [NodeJS](https://nodejs.org){:target="_blank"} (A JavaScript runtime)
* [Selenium JavaScript language bindings](http://docs.seleniumhq.org/download/){:target="_blank"} (But we'll use NPM to grab these)
* [MochaJS](http://mochajs.org/){:target="_blank"} (Mocha is a JS test framework)
* [ChaiJS](http://chaijs.com/){:target="_blank"} (Chai is an assertion library)
* Any browsers installed that you want to test [Supported Platforms](http://docs.seleniumhq.org/about/platforms.jsp){:target="_blank"}

### Selenium Prep
If you haven't read through my quick [Overview of Selenium]({% post_url 2016-06-25-selenium-overview-setup %}) you should do that now. Selenium will need a few things configured before it'll do its magic!

### NodeJs Tutorial
If you're unfamiliar with NodeJS I recommend this tutorial to get you started: [NodeHero](https://blog.risingstack.com/node-hero-tutorial-getting-started-with-node-js/){:target="_blank"}. It's packed with helpful information. It's a good resource to keep open in a tab while working through this post.

### Create Project
To kick things off we'll need a new project. Using a command prompt or Explorer we need to create a new folder for our project. Once we have our folder we can use the command prompt to begin installing the packages we'll need.

### Initializing  Project
NodeJS comes with its own package manager: NPM. It is also used to initialize a project. Run the following command and fill in each section: `npm init`.
This will create the **package.json** file. This file tracks dependencies and project information.

### Installing Packages
NPM is also responsible for installing packages. The command for installing a package is `npm install <package>`. We need to install Selenium, Mocha and Chai.

* `npm install selenium-webdriver --save`
* `npm install mocha --save`
* `npm install chai --save`

These will install in the **node_modules** folder and get tracked in our **package.json** file.

Here's what our folder structure looks like:
![Folder structure]({{ site.url }}{{ site.baseurl }}/assets/images/js-selenium-mocha/structure.png "Folder structure")

We can now write our first test and use our installed packages! Here's what the package.json file should look like at this point:

```json
{
"name": "selenium-js-mocha",
"version": "1.0.0",
"description": "A project to go along with blog posts explaining how to get started with functional testing using Selenium, JavaScript and Mocha. ",
"main": "test1.js",
"dependencies": {
"chai": "^3.5.0",
"mocha": "^2.5.3",
"selenium-webdriver": "^2.53.2"
},
"devDependencies": {},
"scripts": {
"test": "mocha test1.js"
},
"author": "Stephen Cavender",
"license": "ISC"
}
```

### Write Test
We'll write our test against [The Internet](http://the-internet.herokuapp.com/){:target="_blank"}[^theinternet].

[^theinternet]: Credit to [Dave Haeffner](http://davehaeffner.com/){:target="_blank"}.

Let's create a new JS file in our project folder: I'll create test1.js.

```javascript
// Load dependecies
var assert = require('chai').assert,
test = require('selenium-webdriver/testing'),
webdriver = require('selenium-webdriver');

// Our test
test.describe('Test', function () {
test.it('Title should be "The Internet"', function () {
// Set timeout to 10 seconds
this.timeout(10000);

// Get driver
// var driver = new webdriver.Builder().
// withCapabilities(webdriver.Capabilities.firefox()).
// build();
// var driver = new webdriver.Builder().
// withCapabilities(webdriver.Capabilities.edge()).
// build();
// var driver = new webdriver.Builder().
// withCapabilities(webdriver.Capabilities.ie()).
// build();
var driver = new webdriver.Builder().
withCapabilities(webdriver.Capabilities.chrome()).
build();

// Go to URL
driver.get('http://the-internet.herokuapp.com');

// Find title and assert
driver.executeScript('return document.title').then(function(return_value) {
assert.equal(return_value, 'The Internet')
});

// Quit webdriver
driver.quit();
});
});
```

1. Load dependencies
2. Create test
3. Set timeout (test runs too fast and fails without increasing the timeout)
4. Get a web driver
5. Set the URL property (tells the driver to go to that URL)
6. Assert on the title of the driver
7. Dispose of the driver

### Run Test
Now that we have a functional test we can run it. Save the test file and let's grab our command prompt. To run tests we call the mocha command and our test file.

`mocha test1.js`

Run that in the command prompt and we should see our test run and the command prompt will tell us the result of the test. If the **package.json** is set up with a test script we can also run our test by calling the npm test script.

`npm test`

This is how the command prompt displays our test run:
![Command prompt test run]({{ site.url }}{{ site.baseurl }}/assets/images/js-selenium-mocha/npmTestCmd.png "Command prompt test run")

And this is how bash displays our test run:
![Bash test run]({{ site.url }}{{ site.baseurl }}/assets/images/js-selenium-mocha/npmTestBash.png "Bash test run")

This is a basic, and brittle, example of how Selenium works. We'll cover a much better testing approach in a later post to avoid such things! This is not an example of best practices by any means. This is to get you a working example of Selenium. Stay tuned for more posts on how to use Selenium and best practices for automating tests!

Thanks for reading! I hope you found it helpful!
