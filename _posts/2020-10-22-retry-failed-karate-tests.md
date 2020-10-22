---
title: Automatically Retrying Failed Karate Tests
date: 2020-10-22
modified: 2020-10-22
categories: [karate, automated test]
excerpt: Make test execution less flaky by retrying (only) failed tests.
tags: [karate, automated test]
classes: wide
header:
  overlay_image: /assets/images/karate-retry/retry-header.jpg
  overlay_filter: 0.7 # same as adding an opacity of 0.5 to a black background
---

## Karate Tests Prep
If you haven’t read through my article [Powerful GraphQL Automated Test Using Karate](https://dnomyar.dev/karate/automated%20test/karate-graphql){:target="_blank"}, you should do that now. You will need to know how Karate test works firsthand before proceeding with this one. This post will walk you on how to automatically retry failed tests on karate.

## Problem
Karate is an awesome test automation library, there is no denying that. However it currently does not support auto retrying test cases that failed in the first execution. I am talking about something like [this](https://medium.com/@sonaldwivedi/how-to-rerun-only-failed-testcases-using-testng-a23802f6884){:target="_blank"} via TestNG or [this](https://medium.com/@omurdenden/re-run-failed-automated-test-cases-in-robot-framework-jenkins-setup-5d293ea40947){:target="_blank"} via RobotFramework. The closest available mean of retry in karate comes in form of [`retry until`](https://github.com/intuit/karate/releases/tag/v0.9.0){:target="_blank"} of which you need to put manually on all of your test cases.

Personally I understand the reason behind this, as tests should be deterministic. However, while that might work out well in an ideal world; we run our test cases on staging environment (which is far cry from production environment). Sometimes test fail because of random things such as network condition, some services occassionally chug, and many other things.

Our automated tests become too flaky. Considering that we put our tests as a mandatory pipeline to introduce code to staging, it is totally unacceptable having to re-run tests all the time. It becomes more frustrating when every times you re-run the tests, different causes of failure block the pipeline.

![Meme]({{ site.url }}{{ site.baseurl }}/assets/images/karate-retry/bruh.jpeg "Bruh"){:height="75%" width="75%"}


![Meme]({{ site.url }}{{ site.baseurl }}/assets/images/karate-retry/devops.jpg "Bruh"){:height="75%" width="75%"}

**Somehow it turns into this kind of joke for us**

Oopsie, pretty sure it's a pain in the ass and we want to avoid that.

## Solution

I decided to do a research about karate hooks and remembered that you can actually implement karate's `ExecutionHook` and override each of the hooks:
- `beforeScenario`
- `afterScenario`
- `beforeFeature`
- `afterFeature`
- `beforeAll`
- `afterAll`
- `beforeStep`
- `afterStep`
- `getPerfEventName`
- `reportPerfEvent`

Having previously played with these hooks in order to [automatically generates GraphQL queries](https://dnomyar.dev/karate/automated%20test/graphql/auto-generate-grapql-queries-karate/){:target="_blank"}, I found out that it is also possible for us to retry tests utilizing one of these hooks.

Particularly we can override `afterScenario`. We can do these things:
- Add `@retry` tag to failed tests
- Retry the tests with tag `@retry`
- Delete the `@retry` tag from the tests if they are now passing

Below is the code( You can find it along with the whole tests sample [here](https://github.com/dnomyar90/football-karate-demo-graphql/tree/master/qa){:target="_blank"})

```

    @Override
    public void afterScenario(ScenarioResult result, ScenarioContext context) {
        File f = null;
        f = context.rootFeatureContext.feature.getPath().toFile();
        String filePath = f.toString();
        int line = 0;
        Path path = Paths.get(filePath);

        if (result.isFailed()) {
            try {
                String annotation = Files.readAllLines(path).get(line);
                if (!annotation.contains("@retry")) {
                    List<String> lines = Files.readAllLines(path, StandardCharsets.UTF_8);
                    lines.add(line, "@retry");
                    Files.write(path, lines, StandardCharsets.UTF_8);
                }

            } catch (Exception e) {
                e.printStackTrace();
            }
        } else {
            try {
                Charset charset = StandardCharsets.UTF_8;
                String content = new String(Files.readAllBytes(path), charset);
                String pattern = "@retry";
                content = content.replaceAll(pattern, "");
                Files.write(path, content.getBytes(charset));
            } catch (Exception e) {
                e.printStackTrace();
            }

        }
    }
```

And on the test runner part you can simply just do this:

```
public class TestRunner {
    @Test
    public void testParallel() throws Exception {
        String retryPath = System.getProperty("user.dir") + "/target/test-classes/features";
        int retryCount = 3;
        // Run parallelly and retry according to retry count
        Results result = Runner.path("classpath:features").hook(new KarateExecutionHook()).tags("~@ignore")
                .parallel(10);

        if (result.getFailCount() > 0) {
            for (int i = 0; i < retryCount; i++) {
                System.out.println("====Retrying test====");
                result = Runner.path(retryPath).hook(new KarateExecutionHook()).tags("@retry","~@ignore").parallel(10);
                if(result.getFailCount() == 0)
                break;
            }
        }
        Assert.assertTrue(result.getErrorMessages(), result.getFailCount() == 0);
    }
}
```

It is just simply a string manipulation where we append `@retry` tag to failed tests' generated files. Then re-run the tests with `@retry` tag from that folder. Lastly, delete the `@retry` tag if the tests are now passing. Below is an example of the retry process:

![Pipe1]({{ site.url }}{{ site.baseurl }}/assets/images/karate-retry/pipe1.png "Pipe1"){:height="50%" width="50%"}

**- First run: Failed with 51 failing features (68 scenarios)**

_*We are still running all the scenarios on the failing tests, hence on the next execution the number of scenarios we run is not 65 but 68_ ✌️


![Pipe2]({{ site.url }}{{ site.baseurl }}/assets/images/karate-retry/pipe2.png "Pipe2"){:height="50%" width="50%"}

**- Second run: Re-runing 68 failing scenarios. Now only 11 is still failing**


![Pipe3]({{ site.url }}{{ site.baseurl }}/assets/images/karate-retry/pipe3.png "Pipe3"){:height="50%" width="50%"}

**- Last run: 11 remaining test is now passing**

![Pipe1]({{ site.url }}{{ site.baseurl }}/assets/images/karate-retry/iu.gif "Pipe1"){:height="50%" width="50%"}

**Yeay test execution becomes less flaky. Everybody now have time to play Mobile Legends instead of anxiously waiting for test results.**

## Conclusion
It is truly amazing how such simple tweak can save you from a lot of headache. And again this shows how adjustable karate is to your needs, you can do whatever you want with the provided features of the library. However this approach is far from perfect as it still re-runs all of the failing scenarios on a feature.

For example if you have a test feature with 3 scenarios. Using this approach, even though there is only 1 failing scenario on that test feature, we still re-run all of the 3 cases. We are still exploring on how to tweak the approach to handle the re-run more efficiently 🤞

That's all for this post. I hope you can utilise the same (if not better) approach in order to make your test executions more reliable. Please do comment if you have something to ask for discuss (You need to login via FB first though).

_Stay safe, stay healthy, and always wear your masks outside!_

# [Repository](https://github.com/dnomyar90/football-karate-demo-graphql/tree/master/qa){:target="_blank"}