---
title: 'GraphQL Exploration'
date: '2020-04-05T15:00:00.000Z'
---

_Following is an internal blog post I wrote up for my team at Target to convince the other engineers that we should adopt GraphQL. We were in the very early stages of a re-platforming of our internal CMS and were deciding on technologies to use._

Over the past couple of weeks, I've been playing around with GraphQL, a specification for fetching data that is an alternative to REST.  In this post, I'd like to do a light comparison between GraphQL and REST, describe issues that we face on the client in regards to data fetching and provide examples of a functioning application that's built around the Slingshot domain in an effort to display the benefits that GraphQL can afford us.

In short, GraphQL takes a "client-first" approach to data fetching where the client specifies exactly what data it needs via a GraphQL query (think SQL), sends this query to a GraphQL server and the server returns exactly that data back to the client.

Compare this to REST, which is more "server focused".  A service that exposes endpoints that follow REST principals are rigid and generally follow a pattern of:

- GET calls that return a single entity or a list of entities
- PUT/PATCH calls that modify that entity
- POST calls that create that entity
- DELETE calls that delete that entity.

Entities that have relationships with other entities (either via collections or a single property) might return those child entities fully hydrated in the GET call or they might just return a pointer (either a url to get that entity or an id) to the child entity.  A GET endpoint for a single entity will generally return all fields for that entity.

These characteristics of REST present a few problems from the client's perspective:

- A client could make a request for a list of entities, but only really need a subset of fields on that entity, resulting in an "overfetching" of data
- If a client needs to get an entity AND child entities tied to it but the GET endpoint only returns pointers to the child entity, the client will need to orchestrate subsequent API calls to fetch the data for those child entities
- There are zero assurances from the server that the data the client expects is what the server is actually returning.
- A client that requires data from multiple services needs to know about all of those services, which results in the client being exposed and therefore coupled to your underlying service architecture

The above friction points from the client perspective could be somewhat alleviated by introducing a service that exposes custom, non REST endpoints that are specific to the client's needs, however:

- a "custom endpoint" approach might lead to duplication and confusion if there are multiple similar but ultimately different data requirements from the client
- a solution to avoid the above point might be building custom endpoints that return enough data to satisfy all of the aforementioned similar-but-different requests, but will likely result in overfetching from ALL data calls made by the client
- there are still no assurances of the data contract between client and server

These issues are greatly exacerbated as more and more services are required by the client to function.  It's worth noting that Slingshot UI suffers from each problem noted above.

## How does GraphQL help?

GraphQL alleviates much of the friction with REST outlined above as well as offers additional features that help improve developer experience:

- Aggregation layer that sits on top of existing service architecture, insulating the client from service changes as well as greatly reducing the amount of complexity on the client related to fetching data.
- Schema Definition and Type enforcement between client and server
- A single service and endpoint for fetching data from the client

## What is required to integrate GraphQL with an application

It's important to note that GraphQL is a specification, not a specific piece of software, per se.  As such, there are implementations of GraphQL servers in all popular server stacks (JVM, .NET, NodeJS).  To integrate GraphQL in your platform, you'll need:

**GraphQL server** - there are numerous implementations of GraphQL servers.  This GraphQL server can sit on top of existing REST services or have direct access to your data stores.  In the app referenced below, I've chosen Apollo due to their top notch documentation and the impression that they're the current leaders in GraphQL implementations.
**GraphQL client** - depending on your view library (e.g. React), there are various options for a GraphQL client.  I've chosen Apollo Client (specifically apollo-boost) which integrates with React quite well.

That's it!  GraphQL provides for authentication, tracing and all other functionality you'd expect for a service.

## Why introduce GraphQL to Slingshot

I believe that GraphQL will greatly simplify our client code and unlock some productivity boosts when implementing new features.  Most of the friction I've experienced when introducing new features is related to wiring up data on the client.  This work requires a lot of boilerplate related to redux:

- setup actions to fetch data, reducers to process this data and selectors to expose this data to React components.  This often involves touching 3 or 4 files and a ton of indirection
  for data calls that require child entities to be hydrated on the client (e.g. fetching treatment info tied to a given test),
- there is a lot of coordination and N + 1 queries that need to be orchestrated on the client.  This code is not trivial and as such, requires tests to be written against them.  This distracts the developer from focusing on building a UI

Due to the above points, much of the code in Slingshot UI is made up of logic related purely to data fetching and orchestration:

| All Code              | LOC    | %    |
| --------------------- | ------ | ---- |
| Total Lines of Code   | 23,900 | 100  |
| Data Fetching Related | 8,463  | 35.4 |

Over a third of our codebase is related to fetching data for the application.  Digging into those LOC related to data fetching:

| Data Fetching Related Code | LOC   |
| -------------------------- | ----- |
| Test Code                  | 5,214 |
| Redux                      | 5,024 |
| Redux Test Code            | 3,192 |

As mentioned above, there is some complexity inherent to data fetching and orchestration and this has resulted in a bunch of test writing to ensure this behavior is correct.  We can see this in the number of LOC in test files related to data fetching.  Similarly, Redux is quite verbose and can get complicated quickly.  We can see this in the LOC as Redux code comprises over 20% of our codebase, with over 60% of that code being test files.

GraphQL would eliminate the need for nearly all of this code as data fetching logic would be moved from the client to a GraphQL server.  It's important to note that Redux will likely not be needed if we migrate to GraphQL.

<!-- RedOak Page Viewer - GraphQL POC Application
To play around with GraphQL on the server and client, I added a feature to the admin-app that allows the user to search for a page in redoak and view information (slot information, component status, slot overrides) for a specific page slice.  This application consists of a server and client:

redoak-graphql-server - GraphQL server that the admin-app's "page" feature depends on.  Follow the instructions in the readme to run the application locally
admin-app - I've created a branch that includes a "Page"...page that contains the features mentioned above.  Note that this feature is on the "page-POC" branch 
This example puts the features inherent to GraphQL (outlined above) on full display.  In short, the Page view shows data that spans multiple services and resources:

RedOak
component information including statuses, name and scheduling dates. 
page refreshes, including slot information
JIRA
component statuses
Sapphire
test and treatment information
Search
allows access to our "omni-search" endpoint that we leverage in Slingshot
While this data is ultimately utilized by the client, the client has no idea which system the data came from, nor does it care. It simply issues a query to the GraphQL server detailing all the fields it needs.  The client has virtually zero logic related to fetching data, aside from code that describes the GraphQL query that is sent to the server. The server handles this query and performs the aggregation and orchestration in a reusable and elegant fashion.

It is worth mentioning that, despite the fact this feature is not 100% fleshed out, it does include quite a bit of data and somewhat mirrors the UI on the page planner page in Slingshot UI.  And it achieves this with much, much less code:

the client feature is composed of 9-10 files, mostly React components
the service is composed of 4 files
neither the client nor the server contain tests . Though looking at the code, testing these features both on client and server should be trivial.
A big takeaway from the above information is that a relatively rich application feature can be achieved with a small amount of code. And while we do need both a client and server to achieve this feature, there is a better separation of concerns in the application stack where the client can focus purely on UI rendering and the server focuses on responding to a request and fetching/aggregating only the data the client needs based on said request. -->

## Why shouldn't we introduce GraphQL to Slingshot

I don't know! I'm still in the early phases of learning about the technology.  My plan is to meet with a couple engineers at Target that use GraphQL in their products and get their feedback and experiences on the technology.  Based on this feedback and further investigation into the tech, there might be additional reasons not to switch to GraphQL

## Next Steps

I've shared some of this information with Marc and Joe to get their input from a services perspective.  They are looking at Spring implementations to see how it would fit in our stack. I'd like to demo the above admin app feature to the team to show how GraphQL is used on the client and how it can potentially simplify certain aspects of Slingshot.  Based on how everyone's feeling we can flesh out a plan of attack on migrating to GraphQL and how it relates to our 2019 goals related to replatforming Slingshot.
