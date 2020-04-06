---
title: 'GraphQL Exploration'
date: '2020-04-05T15:00:00.000Z'
---

_Following is an internal blog post from 2019 that I wrote up for my team at Target to convince the other engineers that we should adopt GraphQL. We were in the very early stages of a re-platforming of our internal CMS and were deciding on technologies to use. We decided to move forward with GraphQL which was 100% the right decision given our application's needs_

_I removed references to our internal CMS's name and dependent services, hence the generic use of "CMS" and "Service X", "Service Y", etc_

Over the past couple of weeks, I've been playing around with GraphQL, a specification for fetching data that is an alternative to REST.  In this post, I'd like to do a light comparison between GraphQL and REST, describe issues that we face on the client in regards to data fetching and provide examples of a functioning application that's built around the CMS domain in an effort to display the benefits that GraphQL can afford us.

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
- A client that requires data from multiple services needs to know about all of those services, which results in the client being exposed and therefore coupled to the underlying service architecture

The above friction points from the client perspective could be somewhat alleviated by introducing a service that exposes custom, non REST endpoints that are specific to the client's needs, however:

- a "custom endpoint" approach might lead to duplication and confusion if there are multiple similar but ultimately different data requirements from the client
- a solution to avoid the above point might be building custom endpoints that return enough data to satisfy all of the aforementioned similar-but-different requests, but will likely result in overfetching from ALL data calls made by the client
- there are still no assurances of the data contract between client and server

These issues are greatly exacerbated as more and more services are required by the client to function.  It's worth noting that the current CMS UI suffers from each problem noted above.

## How does GraphQL help?

GraphQL alleviates much of the friction with REST outlined above as well as offers additional features that help improve developer experience:

- Aggregation layer that sits on top of existing service architecture, insulating the client from service changes as well as greatly reducing the amount of complexity on the client related to fetching data.
- Schema Definition and Type enforcement between client and server
- A single service and endpoint for fetching data from the client

## What is required to integrate GraphQL with an application

It's important to note that GraphQL is a specification, not a specific piece of software, per se.  As such, there are implementations of GraphQL servers in all popular server stacks (JVM, .NET, NodeJS).  To integrate GraphQL in your platform, you'll need:

**GraphQL server** - there are numerous implementations of GraphQL servers.  This GraphQL server can sit on top of existing REST services or have direct access to your data stores.  In the app referenced below, I've chosen Apollo due to their top notch documentation and the impression that they're the current leaders in GraphQL implementations.

**GraphQL client** - depending on your view library (e.g. React), there are various options for a GraphQL client.  I've chosen Apollo Client (specifically apollo-boost) which integrates with React quite well.

That's it!  GraphQL provides for authentication, tracing and all other functionality you'd expect from a service.

## Why introduce GraphQL to our CMS

I believe that GraphQL will greatly simplify our client code and unlock some productivity boosts when implementing new features.  Most of the friction I've experienced when introducing new features is related to wiring up data on the client.  This work requires a lot of boilerplate related to redux:

- setup actions to fetch data, reducers to process this data and selectors to expose this data to React components.  This often involves touching 3 or 4 files and a ton of indirection
  for data calls that require child entities to be hydrated on the client
- there is a lot of coordination and N + 1 queries that need to be orchestrated on the client.  This code is not trivial and as such, requires tests to be written against them.  This distracts the developer from focusing on building a UI

Due to the above points, much of the code in the CMS UI is made up of logic related purely to data fetching and orchestration:

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

GraphQL would eliminate the need for nearly all of this code as data fetching logic would be moved from the client to a GraphQL server.

It's important to note that Redux will likely not be needed if we migrate to GraphQL.

## GraphQL POC Application

To play around with GraphQL on the server and client, I added a feature to our admin app that allows the user to search for a page in publishing service and view information (slot information, component status, slot overrides) for a specific page slice.  This application consists of a server and client:

- graphql-server - GraphQL server that the admin-app's "page" feature depends on.

- admin-app - I've created a branch that includes a "Page"...page that contains the features mentioned above.

This example puts the features inherent to GraphQL on full display.  In short, the Page view shows data that spans multiple services and resources:

- Publishing
- Component information including statuses, name and scheduling dates.
- Page refreshes, including slot information
- A/B Testing
- Site Search

While this data is ultimately utilized by the client, the client has no idea which system the data came from, nor does it care. It simply issues a query to the GraphQL server detailing all the fields it needs.  The client has virtually zero logic related to fetching data, aside from code that describes the GraphQL query that is sent to the server. The server handles this query and performs the aggregation and orchestration in a reusable and elegant fashion.

It is worth mentioning that, despite the fact this feature is not 100% fleshed out, it does include quite a bit of data and closely mirrors the current CMS UI.

And it achieves this with much, much less code:

-the client feature is composed of 9-10 files, mostly React components
-the service is composed of 4 files
-neither the client nor the server contain tests . Though looking at the code, testing these features both on client and server should be trivial.

A big takeaway from the above information is that a relatively rich application feature can be achieved with a small amount of code on the frontend. And while we do need both a client and server to achieve this feature, there is a better separation of concerns in the application stack where the client can focus purely on UI rendering and the server focuses on responding to a request and fetching/aggregating only the data the client needs based on said request.

## Conclusion

_This is me, today, commenting on the decision to adopt GraphQL in our platform. Following was not in the original post to my team_

The adoption of GraphQL in our platform rewrite _greatly_ improved the developer experience in the UI as well as allowed a healthy separation between frontend and backend in regards to data fetching. It also promoted an effective approach to new features where UI engineers and backend engineers would discuss schema design and, upon agreement, would go to their respective codebases and develop against this contract.

Adoption of GraphQL was not without its pitfalls, false starts and refactorings; all of which I hope to capture in a future post. However, the benefits far exceeded the drawbacks and allowed us to deliver a ton of useful features in less time than if we continued down the Redux path we were previously on.
