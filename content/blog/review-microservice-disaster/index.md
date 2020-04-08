---
title: 'Review: Microservice Disaster'
date: '2020-04-08T13:32:00Z'
---

[Link to YouTube Video](https://www.youtube.com/watch?v=gfh-VCTwMw8)

## Overview

[Jimmy Bogard](https://jimmybogard.com/) presents a problem that one of his clients ran into when replatforming to a Service Oriented Architecture. He describes how service boundaries should be drawn and provides an example of a refactoring/restructuring of a search service,resulting positive results in performance.

## Key Takeaways 

- data duplication is not, in itself, a bad thing so don’t shy away from it
- disfunctional organizations with perverse incentives will result in poorly architected and crappy systems
- service API calls should not make multiple hops. If a service calls another service, that service shouldn’t call another service
- a search service that depends on an aggregation of data from multiple domains (e.g. Catalog, Orders, etc) can be/should be decoupled from those domain services and should manage the data itself
- related to above, "managing the data itself means" being in charge of fetching updated data on a set schedule or processing events when the underlying domain data changes