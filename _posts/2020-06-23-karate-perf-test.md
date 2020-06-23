---
title: Reusing Karate Tests as Performance Test
date: 2020-06-23
modified: 2020-06-23
categories: [karate, automated test, performance test]
excerpt: Quickly set up your first performance test with karate. No need to write your performance test from scratch. Just reuse your existing tests!
tags: [karate, automated test, performance test]
classes: wide
header:
  overlay_image: /assets/images/reuse-karate-perf/perftest.jpg
  overlay_filter: 0.3 # same as adding an opacity of 0.5 to a black background
  caption: "Photo by [Anton Jansson](https://unsplash.com/@antonjphotos) on [Unsplash](https://unsplash.com)"

---

## Karate Tests Prep
If you haven’t read through my article [Powerful GraphQL Automated Test Using Karate](https://dnomyar.dev/karate/automated%20test/karate-graphql){:target="_blank"}, you should do that now. You will need to know how Karate test works before proceeding with this one 😉

## What is Performance Testing?

Performance testing checks the speed, response time, reliability, resource usage, scalability of a software program under their expected workload. The purpose of Performance Testing is not to find functional defects but to eliminate performance bottlenecks in the software or device.

![Graph]({{ site.url }}{{ site.baseurl }}/assets/images/reuse-karate-perf/graph.jpg "Graph"){:height="50%" width="70%"}

<sub align="center">
[Illustration](https://unsplash.com/photos/0jTZTMyGym8){:target="_blank"} of performance testing
</sub>

The focus of Performance Testing is checking a software program's:

- Speed - Determines whether the application responds quickly
- Scalability - Determines maximum user load the software application can handle.
- Stability - Determines if the application is stable under varying loads


<sub>*Quoted from [Guru99](https://www.guru99.com/performance-testing.html){:target="_blank"}[^guru99]</sub>

[^guru99]: Credit to [Guru99](https://www.guru99.com/performance-testing.html){:target="_blank"}.

## Performance Test With Karate
[Karate](https://intuit.github.io/karate/){:target="_blank"} provides the ability to reuse your existing karate API tests as performance test. This means that you do not need to write your performance test from scratch. All the assertions and scenarios on your existing tests can be reused!

We will learn how to set up and run karate tests as performance test. All the code can be found on the [repository](https://github.com/dnomyar90/football-karate-demo-graphql/tree/master/qa){:target="_blank"}. Let's get started!


For the sake of providing context, below is how the karate tests we use for performance test will look like:

```
@mutation
Feature: Test GraphQL Insert Player (mutation)

    Background:
        Given url baseUrl

    Scenario: Insert a player

    # Read GraphQL file
    Given def query = read('graphql/insertplayer.graphql')

    # Declare random variables for the request (in JSON format)
    And def variables = {last_name: '#(faker.randomLastName())',first_name:'#(faker.randomFirstName())', team_id:2}

    # Prepare the request
    And request { query: '#(query)', variables: '#(variables)' }
    When method POST
    Then status 200

    # Parse string as integer & assert with existing variable
    And match response.data.player.first_name == variables.first_name
    And match response.data.player.last_name == variables.last_name

    # Assert that there is no error
    And match response.errors == '#notpresent'
```

Before we proceed to write our performance test, we need to add dependecy and plugin on our `pom.xml`. There is no need to worry, for everything has been added on the [repository](https://github.com/dnomyar90/football-karate-demo-graphql/blob/master/qa/pom.xml){:target="_blank"} 😄

**Dependency:**
```
 <dependency>
        <groupId>com.intuit.karate</groupId>
        <artifactId>karate-gatling</artifactId>
        <version>0.9.5.RC3</version>
        <scope>test</scope>
</dependency>
```

**Plugin:**
```
  <plugin>
	<groupId>net.alchim31.maven</groupId>
	<artifactId>scala-maven-plugin</artifactId>
	<version>4.3.1</version>
	<executions>
		<execution>
			<id>scala-test-compile</id>
			<phase>process-test-resources</phase>
			<goals>
				<goal>testCompile</goal>
			</goals>
		</execution>
	</executions>
</plugin>
<plugin>
	<groupId>io.gatling</groupId>
	<artifactId>gatling-maven-plugin</artifactId>
	<version>3.0.5</version>
	<configuration>
		<simulationsFolder>src/test/java/simulations</simulationsFolder>
	</configuration>
</plugin>
```

The dependency and plugin above will enable us to write scala tests on top of existing karate features. Writing scala performance test is relatively straightforward. Below is the sample of a performance test on scala. Again no need to worry, all of these can be found on the [repository](https://github.com/dnomyar90/football-karate-demo-graphql/tree/master/qa){:target="_blank"}

```
package perftest

import com.intuit.karate.gatling.PreDef._
import io.gatling.core.Predef._
import scala.concurrent.duration._

class PerfTests extends Simulation {

  before {
    println("Perf tests started")
  }
  
  // Amount of ramp users
  val amountRampUsers: Int = 10

  // Duration of ramp
  val amountRampDuration: Int = 10

  val playersTestQuery = scenario("Query Players")
    .exec(
      karateFeature("classpath:features/FindPlayers.feature")
    )

    val playersTestMutation = scenario("Mutation Players")
    .exec(
      karateFeature("classpath:features/InsertPlayer.feature")
    )

  setUp(
    playersTestMutation.inject(rampUsers(amountRampUsers) during amountRampDuration),
    playersTestQuery.inject(rampConcurrentUsers(1) to (5) during amountRampDuration)
    ).assertions(
        global.responseTime.max.lt(500),
        global.successfulRequests.percent.gt(95)
    )

  after {
    println("Perf tests ended")
  }

} 
```

Quite straightforward right? The scala script executes `FindPlayers` and `InsertPlayer` tests by injecting two different scenarios. For `InsertPlayer` we inject 10 ramp users in 10 seconds duration. On the other hand for `FindPlayers` we inject 1 to 5 concurrent ramp users in 10 seconds duration. We then assert that the maximum response time is 500ms (_The sample GraphQL apps is indeed very bad in performance_ 🤣) and success rate of endpoints are at 95%.

> What is ramp users?
>
> What is concurrent ramp users?

**Great question!**

For the `InsertPlayer` test, we set maximum numbers of users accessing the endpoint to 10 users in 10 seconds duration. Meanwhile for the `FindPlayers` we set concurrent users accessing the endpoint between 1 to 5 users for 10 seconds. You can find the whole documentation of gatling [here](https://gatling.io/docs/current/general/simulation_setup){:target="_blank"}. You can have all combinations for assertion that suits your need for performance testing. All of scala injection and assertion is quite straightforward and should not be hard to implement!

We then can run the scala test we have written by running this command:

```
mvn clean test-compile gatling:test
```

Or if you have clone the [repository](https://github.com/dnomyar90/football-karate-demo-graphql/){:target="_blank"}, you can just go to folder `qa` and run `./run-perf-test.sh`

```
cd qa
./run-perf-test.sh
```


## Performance Test With Karate -- Result

- The test will trigger success if all the assertion on our scala test script is fulfilled. Otherwise it will signal an error.

![Console]({{ site.url }}{{ site.baseurl }}/assets/images/reuse-karate-perf/gatling-console.png "Console"){:height="90%" width="90%"}
<p align="center">
<i>Console execution result</i>
</p>

- **Execution result of our performance test**. It shows all information you need in brief. The information includes request per seconds, response time percentiles, number of success and failed requests, etc. This is typically good for your overview dashboard.

![Overview]({{ site.url }}{{ site.baseurl }}/assets/images/reuse-karate-perf/gatling-overview.png "Overview"){:height="90%" width="90%"}
<p align="center">
<i>Overview execution result</i>
</p>


- **Detailed execution result of our performance test**. It shows the detailed information including response time percentiles, response time distribution, number of responses per second, etc.

![Detailed]({{ site.url }}{{ site.baseurl }}/assets/images/reuse-karate-perf/gatling-response-time.png "Detailed"){:height="90%" width="90%"}
<p align="center">
<i>Detailed execution result</i>
</p>

Alright, that's a wrap. Thanks for reading! I hope you find this it useful and you can implement it to increase the overall quality of your product.



# [Repository](https://dnomyar.dev/karate/automated%20test/karate-graphql){:target="_blank"}