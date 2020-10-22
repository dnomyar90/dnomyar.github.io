---
title: Reusing Fragments and Autogenerate GraphQL Queries Using Karate
date: 2020-07-25
modified: 2020-07-25
categories: [karate, automated test, graphql]
excerpt: Make your GraphQL testing more enjoyable. Autogenerate your GraphQL queries & reuse fragments modularly straight from your automated tests!
tags: [karate, automated test, graphql]
classes: wide
header:
  overlay_image: /assets/images/karate-gql/gql-karate.jpg
  overlay_filter: 0.7 # same as adding an opacity of 0.5 to a black background
  caption: "Photo by [svklimkin](https://unsplash.com/@svklimkin) on [Unsplash](https://unsplash.com)"

---

## Karate Tests Prep
If you haven’t read through my article [Powerful GraphQL Automated Test Using Karate](https://dnomyar.dev/karate/automated%20test/karate-graphql){:target="_blank"}, you should do that now. You will need to know how Karate test works firsthand before proceeding with this one. This post will walk you through the workaround to organise your GraphQL queries and fragments using karate.

## Problem Statement
As I have mentioned on my other [article](https://dnomyar.dev/karate/automated%20test/karate-graphql){:target="_blank"}, karate is really great for either REST or [GraphQL](https://graphql.org/) requests. To be speicific in my current company ([kumparan.com](https://kumparan.com/)), we are dealing with GraphQL. GraphQL test automation is provided out of the box with karate. Based on the example on karate's github, you can either use plain text or even read from GraphQL files.

GraphQL request looks something like this:

```
query FindAllPlayers {
FindAllPlayers {
...Player
}
}

fragment Player on Player {
id
first_name
last_name
team{
...Team
}
}

fragment Team on Team {
id
name
is_league_winner
current_competitions
league{
...League
}
}

fragment League on League {
id
name
continent
}
```

AFAIK this is the standard way of doing things in tools like [Postman](https://www.postman.com/) or [Insomnia](https://insomnia.rest/). We need to write each fragment again and again for each query. This generates two big concerns for me:
- If a fragment is changed, we need to find and replace each fragment on each query
- Writing queries to use in Postman or Insomnia will be a painful process

## Solution for problem #1

> ***Problem #1***:
> "If a fragment is changed, we need to find and replace each fragment on each query"

We decided that we must design the usage of fragments and queries on our karate tests to be as modular as possible. We achieve our goal by separating fragments and queries. We declare them in their own specific javascript files. Karate really comes in handy as it can easily make javascript calls out of the box. 

We ended up using this kind of setup:

```
-- feature files
  -- FindAllPlayers.feature
  -- FindPlayerByID.feature
  -- AddNewPlayer.feature
  -- ....
-- queries
  -- FindAllPlayers.js
  -- FindPlayerByID.js
  -- ....
-- mutations
  -- AddNewPlayer.js
  -- ....
   -- fragments
      -- FragmentPlayer.js
      -- FragmentLeague.js
      -- FragmentTeam.js
      -- ....
```

Each `.feature` is going to call `queries` or `mutations`. On each `query` or `mutation` they are going to specify the operation to use and the fragments they need. Hence, no need for us to hardcode each fragment to every query or mutation.

Below are the code implementation of the setup (which you can find in the [repository](https://github.com/dnomyar90/football-karate-demo-graphql/tree/master/qa){:target="_blank"} too):
### Feature file
In feature file we call the javascript file for query or mutation. It is as simple as that.

```
@query
Feature: Test GraphQL Find Players

    Background:
        Given url baseUrl

    Scenario: Find All Players

      Given def query = karate.call('classpath:queries/FindAllPlayers.js')
      
      And request { query: '#(query)'}

```

### Query file
Here we declare the query with all of it's parameters. We call the fragments we need.

`helper.js` is a string manipulation javascript file we use to _flatten_ the fragments. It works to avoid duplicated fragments to be declared (sometimes in complex requests we call a lot of fragments). Of course you can find it on the [repository](https://github.com/dnomyar90/football-karate-demo-graphql/tree/master/qa){:target="_blank"} too.
```
function () {
    var query =
        'query FindAllPlayers {\
            FindAllPlayers {\
                    ...Player\
                }\
                }';

    var fragments = [
        karate.call('classpath:fragments/FragmentPlayer.js').Player()
    ];
    return karate.read('classpath:helper.js')(query, fragments);
}
```

### Fragment file
Here we declare the smallest unit of all tests. Each fragment is declared in a modular fashion that we will return them in form of array. Hence, making it possible for us to reuse fragments from another files.

```
//FragmentPlayer.js
function() {
    var Player =
      'fragment Player on Player {\
        id\
        first_name\
        last_name\
        team{\
          ...Team\
        }\
     }';
  
    return {
      Player: function () { return [Player,
        karate.call('classpath:fragments/FragmentTeam.js').Team()
      ]
    }
  }
}
```

```
//FragmentTeam.js
function() {
    var Team =
      'fragment Team on Team {\
        id\
        name\
        is_league_winner\
        current_competitions\
        league{\
          ...League\
        }\
     }';
  
    return {
      Team: function () { return [Team,
        karate.call('classpath:fragments/FragmentLeague.js').League()
      ]
    }
  }
}
```

```
//FragmentLeague.js
function() {
    var League =
      'fragment League on League {\
        id\
        name\
        continent\
     }';
  
    return {
      League: function () { return [League]}
  }
}
```

By achieving this setup, we managed to avoid the dangerous part of hard coding fragments to each GraphQL requests. So far it is quite useful for us too in terms of making writing tests a simpler task. When we are about to write new tests with existing fragments, we can just call the existing fragments instead of writing all required fragments from scratch.

## Solution for Problem #2
> ***Problem #2***:
> "Writing queries to use in Postman or Insomnia will be a painful process"

After using karate for GraphQL test automation for some time, we faced another problem (although it is unrelated with karate). We are testing GraphQL requests for new features or refactors using [Insomnia](https://insomnia.rest/).

In Insomnia, like Postman, they have no _object-oriented-approach_ to reuse fragments. So we need to type in fragments all over again for all requests. This increasingly becomes annoying as it hampers productivity. Plus it becomes a risk since we might miss keying in some part of the fragments that may otherwise result in failure when called.

After reading that in karate we can override embedded execution hook, we managed to auto generate all existing requests on karate tests so that we won't need to key in all fragments manually again. It is really fascinating that karate provides both simplicity and at the same time possibility for us to dig deeper and toy around to cater our specific needs.

So basically this is what we do with the test runner:
- Intercept before requesting a query
- Get the query/mutation name
- Generate the `.graphql` file for that query/mutation
- Write the query/mutation content into that `.graphql` file

```
@Override
    public boolean beforeStep(Step step, ScenarioContext context) {
        if (step.getText().trim().contains("request {")) {
            try {
                File dir = new File("GQL-Queries");
                dir.mkdirs();
                String rawQuery = context.vars.get("query").getValue().toString();
                String[] splittedQuery = rawQuery.split("\n", 2);
                String firstLineQuery = splittedQuery[0];
                String queryName = firstLineQuery.replace("query","").replace("mutation", "").replaceAll("\\(.*?\\{", "").replaceAll(" ","").replace("}","").replace("{","")+ ".graphql";
                File file = new File(dir, queryName);
                FileWriter myWriter = new FileWriter(file);
                myWriter.write(context.vars.get("query").getValue().toString());
                myWriter.close();
              } catch (IOException e) {
                e.printStackTrace();
              }
        }
        return true;
    }
```

The above file is available on [repository](https://github.com/dnomyar90/football-karate-demo-graphql/tree/master/qa){:target="_blank"}

The hook above will generate `.graphql` files for each request, even if you call multiple requests in a single `.feature` file. It generates file in the folder `GraphQL-Queries` after each exeuction:

![GQLFiles]({{ site.url }}{{ site.baseurl }}/assets/images/karate-gql/GQLFiles.png "GQLFiles"){:height="75%" width="75%"}

You can then reuse those `.graphql` files on Postman, Insomnia, or any other tools you use:

![FindAllPlayers]({{ site.url }}{{ site.baseurl }}/assets/images/karate-gql/FindAllPlayers.png "FindAllPlayers"){:height="75%" width="75%"}

![AddNewPlayer]({{ site.url }}{{ site.baseurl }}/assets/images/karate-gql/AddNewPlayer.png "AddNewPlayer"){:height="75%" width="75%"}

With the approach above, it is more enjoyable now to test GraphQL requests since we don't have to type requests manually. We just need to look into our [collection](https://github.com/dnomyar90/football-karate-demo-graphql/tree/master/qa/GQL-Queries){:target="_blank"} or run our tests locally. Our karate tests now not only works well for automated test, but also help to increase our productivity when testing features/refactors 😃

That's it! Hope you find it useful. 

_Stay safe, stay healthy, and wear your masks!_ ✌️

# [Repository](https://github.com/dnomyar90/football-karate-demo-graphql/tree/master/qa){:target="_blank"}