---
title: Powerful GraphQL Automated Test With Karate
date: 2020-06-19
modified: 2020-06-19
categories: [karate, automated test]
excerpt: Demo of karate test framework for GraphQL application. Sample GraphQL application is provided.
tags: [karate, automated test]
classes: wide
header:
  overlay_image: /assets/images/karate-graphql/header.jpg
  overlay_filter: 0.2 # same as adding an opacity of 0.5 to a black background
  caption: "Photo by [Jason Briscoe](https://unsplash.com/@jsnbrsc) on [Unsplash](https://unsplash.com)"

---

## What is GraphQL
Most of us probably already know about REST, as REST seems to be the go-to standard for APIs. Some of us may be thinking, what is a GraphQL? How is it different from REST? Well I was thinking the same when I first interviewed at [kumparan](https://kumparan.com/).

I had no idea whatsoever of what GraphQL is about, let alone how to test it. But after dealing with (_testing_) GraphQL server for 1 year, I cannot look at traditional REST API the same way again. GraphQL really solves the over-fetching problem of REST.

The biggest problem with traditional REST is over-fetching. Using REST, we have to call multiple endpoints to get informations we need. I would refer to this great example from [HowToGraphql](https://www.howtographql.com/basics/1-graphql-is-the-better-rest/):

![REST]({{ site.url }}{{ site.baseurl }}/assets/images/karate-graphql/rest.png "REST"){:height="75%" width="75%"}

Let's assume we need to fetch:
- An user's name
- The user's last post's content
- The user's last 3 followers' name

Using REST, we need to make 3 requests to get all the information we need. Those 3 requests also return some information that we not need. Things can get messy real quick especially if some of the endpoints are returning huge amount of information in a request. This is the classic problem of over-fetching in REST.

Using GraphQL, we can specify all information we need to get in a single request. This is because GraphQL is using a declarative approach to fetch data. What in the world is declarative approach? Good question. Again, I will provide the great example from [HowToGraphql](https://www.howtographql.com/basics/1-graphql-is-the-better-rest/):

![GraphQL]({{ site.url }}{{ site.baseurl }}/assets/images/karate-graphql/graphql.png "GraphQL"){:height="75%" width="75%"}

As you can see, we only need to send a single query to GraphQL server in order to get information we need. We do not burden our server with multiple requests. The GraphQL server also efficiently do not fetch informations we do not request for.

Yeay, now you know what GraphQL is and how does it compare to REST. Do not worry. Even if you have not, you can find a sample GraphQL application that you can toy with on my [repository](https://github.com/dnomyar90/football-karate-demo-graphql). The sample application is about football players, teams, and leagues.

In brief, the application contains relational data between players with their respective teams and leagues. You can fetch data of players, leagues, and teams. You can also try to insert players information. The application utilizes PostgreSQL alongside GraphQL server. No need to worry as installation should be really easy to do. I also provided a data migration, so you don't have to setup tables manually on the database 😉

_Below is what the application looks like:_

```
query{
  players{
    id
    first_name
    last_name
    team{
      id
      name
      is_league_winner
      current_competitions
      league{
        id
        name
        continent
      }
    }
  }
}
```

By having the query above, you can get information of players and their detailed information of teams & leagues:

```
{
  "data": {
    "players": [
      {
        "id": "1",
        "first_name": "Messi",
        "last_name": "Lionel",
        "team": {
          "id": 1,
          "name": "Barcelona",
          "is_league_winner": true,
          "current_competitions": [
            "Copa Del Rey",
            "La Liga"
          ],
          "league": {
            "id": 6,
            "name": "La Liga",
            "continent": "Europe"
          }
        }
      },
      {
        "id": "2",
        "first_name": "Henry",
        "last_name": "Thierry",
        "team": {
          "id": 2,
          "name": "Arsenal",
          "is_league_winner": true,
          "current_competitions": [
            "English Premier League",
            "Carling Cup"
          ],
          "league": {
            "id": 1,
            "name": "English Premier League",
            "continent": "Europe"
          }
        }
      }
    ]
  }
}
```

Okay, we have learned how to fetch information using GraphQL. But how do we test GraphQL operations? Great question.

GraphQL is quite different from REST in terms of request's data type. Most of requests and responses on REST is based on JSON or XML. GraphQL on the other hand employs a query that may look like JSON. But it is actually not JSON.

After doing some research, we decided to go with [Karate](https://intuit.github.io/karate/). Karate provides the ability to deal with GraphQL out of the box. At the same time, it provides what I believe to be one of (*if not*) the most powerful assertions in the whole API testing world.

## Karate Test Framework
Karate provides an out of the box support for GraphQL queries and mutations. Karate also supports REST, and even UI Automated Testing including desktop applications. Another very enticing feature is that we can reuse existing Karate API test scenarios as performance test. Yes, you hear it right. I will cover it in another article in the future.

But if you ask me what my most favourite feature of karate is, it would be it's insanely simple yet powerful assertion. GraphQL may be efficient and all that, but it can also gives you headache testing it. It is very likely you will encounter nested response on GraphQL applications. Based on my past experience on using [REST-Assured](http://rest-assured.io/), I already expected that I will have to deal with gruesome task asserting nested data.

**_Except that with Karate, I don't_**

I am going to give you a peek of how easy you can do request and assertion on Karate. All of the examples are available on the [repository](https://github.com/dnomyar90/football-karate-demo-graphql/tree/master/qa). Alright, let's get into it!

```
 Scenario: Find All Players

    # Define GraphQL query
    # In karate you can use multiline declaration

    Given text query =
    """
    {
      players{
        id
        first_name
        last_name
        team{
            id
            name
            is_league_winner
            current_competitions
            league{
                id
                name
                continent
            }
        }
    }
  }
    """

        # Request the query
        And request { query: '#(query)'}

        # Use method POST (GraphQL always uses POST)
        When method POST

        # Assert status code is OK
        Then status 200

        # Print response
        And print response
```

As you might have noticed, Karate employs a cucumber-like syntax. It is however not cucumber, as we do not need to write a _glue_ code to translate the sentences. The example above shows you how we can even use multi-line variable on Karate. Below are the data that will be returned:

```
{
  "data": {
    "players": [
      {
        "id": "1",
        "first_name": "Messi",
        "last_name": "Lionel",
        "team": {
          "id": 1,
          "name": "Barcelona",
          "is_league_winner": true,
          "current_competitions": [
            "Copa Del Rey",
            "La Liga"
          ],
          "league": {
            "id": 6,
            "name": "La Liga",
            "continent": "Europe"
          }
        }
      },
      {
        "id": "2",
        "first_name": "Henry",
        "last_name": "Thierry",
        "team": {
          "id": 2,
          "name": "Arsenal",
          "is_league_winner": true,
          "current_competitions": [
            "English Premier League",
            "Carling Cup"
          ],
          "league": {
            "id": 1,
            "name": "English Premier League",
            "continent": "Europe"
          }
        }
      }
    ]
  }
}
```

Okay, now we head to the interesting part: **assertion**. Below you can find some of assertions that you can do on Karate. Some nested assertions included. 

_*Spoiler: You can do nested assertion just by one line of code on Karate_ 😄

- Asserting response time is less than 100ms:
```
 Then assert responseTime < 100
```

- Asserting each record of player's team is_league_winner is boolean:
```
Then match each response.data.players[*].team.is_league_winner == '#boolean'
```

- Asserting data of players is returned (empty array is okay):
```
Then match response.data.players != '#null'
```

- Asserting each player has a team competing in competitions:
```
Then match each response.data.players[*].team.current_competitions[*] != '#null'
```

- Asserting that there is a team competing in "La Liga" competition:
```
Then match response.data.players[*].team.current_competitions[*] contains "La Liga"
```

- Asserting that there is a team competing in "La Liga" and "Copa Del Rey" competition:
```
And def competitions = ["Copa Del Rey", "La Liga"]
Then match response.data.players[*].team.current_competitions[*] contains competitions
```

- Asserting that there is no team competing in "La Liga" and "Bundesliga" competition:

```
# This is one of karate's own style wildcards
# It might look unfamiliar. But you will get used to it in no time
And def competitions = ["La Liga", "Bundesliga"]
Then match response.data.players[*].team.current_competitions[*] !contains '#(^competitions)'
```

- Assert no error response is returned:
```
Then match response.errors == '#notpresent'
```

I can go on and on, but there are simply a lot of combinations to do. I suggest you try it by yourself using examples provided on my [repository](https://github.com/dnomyar90/football-karate-demo-graphql). You will be amazed on how simple yet powerful some of the assertions are in [Karate](https://intuit.github.io/karate/).


Thanks for reading! I hope you found it useful! 