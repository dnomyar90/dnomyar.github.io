---
title: Selenium Overview & Setup
date: 2016-06-25
modified: 2018-11-15
categories: [Selenium]
excerpt: A quick overview of Selenium and guide on getting it ready for glory!
tags: [selenium]
classes: wide
header:
  overlay_image: /assets/images/selenium-overview/header.jpg
  overlay_filter: 0.5 # same as adding an opacity of 0.5 to a black background
  caption: "Photo by [Philipp Katzenberger](https://unsplash.com/@fantasyflip) on [Unsplash](https://unsplash.com)"
---

## Quick Selenium Overview
In this article we'll explore what Selenium is and how to prep our environment to use it.

> Selenium is an umbrella project for a range of tools and libraries that enable and support the automation of web browsers. - From [Selenium HQ](https://seleniumhq.github.io/docs/index.html){:target="_blank"}

Selenium enables us to:

1. Open browsers
2. Navigate web pages
3. Find web elements
4. Manipulate web elements
5. Run JavaScript

### Language Bindings
First we'll need to get the language bindings we want. The bindings will need to be downloaded and referenced in our project. Some IDE's have package managers that will do this for us. We'll talk more about how that's done in separate articles. For now just know that we'll need some language bindings to get Selenium to work.

### Webdrivers
Selenium's power comes from [webdrivers](http://docs.seleniumhq.org/docs/03_webdriver.jsp){:target="_blank"}; called such because they "drive the web." In the articles to come we'll be using Chrome, Firefox, Edge and IE; we'll need drivers for each.

#### Chrome
[ChromeDriver](https://sites.google.com/a/chromium.org/chromedriver/){:target="_blank"} is a separate executable loaded by Selenium when Chrome is the desired browser. Download the latest driver and place it in an easily-accessible location. I keep mine in `C:/webdrivers/`. Download and install the latest [Chrome](https://www.google.com/intl/en/chrome/browser/desktop/index.html){:target="_blank"}.

#### Firefox
The [FirefoxDriver](https://github.com/SeleniumHQ/selenium/wiki/FirefoxDriver){:target="_blank"} comes built-in with the Selenium language bindings. Download and install the latest [Firefox](https://www.mozilla.org/en-US/firefox/all/){:target="_blank"}.
Mozilla is working on a new driver for their browser called [MarionetteDriver](https://developer.mozilla.org/en-US/docs/Mozilla/QA/Marionette){:target="_blank"}. However, it has not been released as of this writing. Feel free to download the latest driver and put it in the same location as the ChromeDriver.

#### Edge
The [Microsoft WebDriver](https://www.microsoft.com/en-us/download/details.aspx?id=48212){:target="_blank"} is a separate executable loaded up by Selenium when Edge is requested as a browser. Download the latest driver and keep it in the same location as the other driver(s). Make sure you have the latest updates for Edge.

#### IE
The [IEDriverServer](http://docs.seleniumhq.org/download/){:target="_blank"} is maintained and distributed by the Selenium group. Download the latest driver and put it in the same location as the other driver(s). Make sure you have the latest updates for IE.

### Adding to PATH
Now that we have all the drivers we'll want to use we need to add an entry to our PATH environment variable. If you've never done anything with environment variables I would recommend reading this [guide](http://www.howtogeek.com/118594/how-to-edit-your-system-path-for-easy-command-line-access/){:target="_blank"}. Follow the instructions and add the folder containing your webdrivers.

Now Selenium will be ready to go! Check out one of my other articles for the environment you want to use with Selenium:

* [C#, Selenium & MSTest]({{ site.url }}{{ site.baseurl }}/selenium/dotnet-selenium-mstest-quickstart){:target="_blank"}
* [JavaScript, Selenium & Mocha]({{ site.url }}{{ site.baseurl }}/selenium/js-selenium-mocha-quickstart){:target="_blank"}

Thanks for reading! Be sure to share this post if you found it helpful and don't hesitate to chat with me about it!
