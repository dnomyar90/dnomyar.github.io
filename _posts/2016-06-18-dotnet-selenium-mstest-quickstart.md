---
title: C# Selenium MSTest QuickStart Guide
date: 2016-06-18
modified: 2018-11-15
categories: [Selenium]
excerpt: A quick how-to on setting up a solution using MSTest and Selenium!
tags: [Selenium, MSTest, dotnet, windows]
classes: wide
header:
  overlay_image: /assets/images/dotnet-selenium-mstest/header.jpg
  overlay_filter: 0.5 # same as adding an opacity of 0.5 to a black background
  caption: "Photo by [Andras Vas](https://unsplash.com/@wasdrew) on [Unsplash](https://unsplash.com)"

---

## Getting started with C#, Selenium and MSTest!
In this article we'll be using MSTest and Selenium to write tests for web applications. This will be a starter project we can build on for various projects and in future articles.

### Requirements
Here are the requirements before we get started:

 * [Visual Studio](https://www.visualstudio.com/en-us/downloads/download-visual-studio-vs.aspx){:target="_blank"} (A Microsoft Integrated Development Environment or IDE)
 * [Nuget](http://docs.nuget.org/consume/installing-nuget){:target="_blank"} (A package manager for Visual Studio)
 * [Selenium C# language bindings](http://docs.seleniumhq.org/download/){:target="_blank"} (But we'll use NuGet to grab these)
 * Any browsers installed that you want to test [Supported Platforms](http://docs.seleniumhq.org/about/platforms.jsp){:target="_blank"}

### Selenium Prep
If you haven't read through my quick [Overview of Selenium]({% post_url 2016-06-25-selenium-overview-setup %}) you should do that now. Selenium will need a few things configured before it'll do its magic!

### Create Project
To kick things off we'll need a new project.

**File > New > Project**

![Create a new project]({{ site.url }}{{ site.baseurl }}/assets/images/dotnet-selenium-mstest/new-project.png "Create a new project")
Select the Unit Test Project template (**Templates > Visual C# > Test**), give it a name and configure some options. Press Ok to create the project. Visual Studio will create the project and open up your first UnitTest class.
![New project's first view]({{ site.url }}{{ site.baseurl }}/assets/images/dotnet-selenium-mstest/new-project-first-view.png "New project's first view")

### Import/Install Selenium
Now we'll need to grab the Selenium DLLs and give our project access to them.

**Tools > NuGet Package Manager > Manage NuGet Packages for Solution**

![Open NuGet]({{ site.url }}{{ site.baseurl }}/assets/images/dotnet-selenium-mstest/open-nuget.png "Open NuGet")
Select Browse, search for 'Selenium' and install both Selenium.WebDriver and Selenium.Support for your new project.

### Write Test
Now the fun begins; we can write the first Selenium test!
We'll write our test against [The Internet](http://the-internet.herokuapp.com/){:target="_blank"}[^theinternet].

[^theinternet]: Credit to [Dave Haeffner](http://davehaeffner.com/){:target="_blank"}.

Here's some code to put inside the **TestMethod1()**

```c#
//var driver = new OpenQA.Selenium.Firefox.FirefoxDriver
//var driver = new OpenQA.Selenium.Edge.EdgeDriver
//var driver = new OpenQA.Selenium.IE.InternetExplorerDriver
var driver = new OpenQA.Selenium.Chrome.ChromeDriver
{
Url = "http://the-internet.herokuapp.com/"
};
Assert.IsTrue(driver.Title == "The Internet");
driver.Dispose();
```

1. Get a web driver
2. Set the URL property (tells the driver to go to that URL)
3. Assert on the title of the driver
4. Dispose of the driver

### Run Test
Now that we have a functional test we can run it. First, if the Test Explorer isn't displayed we need to add it.

**Test > Windows > Test Explorer**

Our test isn't showing up yet. We need to build the solution for it to recognize that we've written a test it can run. Right click on the solution in the Solution Explorer and Build or Rebuild the solution. If the build is successful we should see our test show up in the Test Explorer. Now we can right click on our test and tell it to run. If all went according to plan we should see a Chrome window pop up, navigate to Google's home page and then close.

This is a basic, and brittle, example of how Selenium works. If our assertion returns false the test will report a failure but the browser window will still be alive. This test is brittle in that it can't run any code after the Assert if the Assert returns false. We'll cover a much better testing approach in a later post to avoid such things! This is not an example of best practices by any means. This is to get you a working example of Selenium. Stay tuned for more posts on how to use Selenium and best practices for automating tests!

Thanks for reading! I hope you found it helpful!
