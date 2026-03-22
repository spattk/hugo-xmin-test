---
title: "REST vs GraphQL"
date: "2022-11-12"
tags: ["api", "graphql", "rest", "backend"]
---

The main difference between them is how data is sent to the client.

## REST Architecture

- The client makes an HTTP request and the data is sent as an HTTP response.
- The structure of the request object is defined on the server.
- REST requires sending multiple API requests to fetch data from multiple sources.
- The problem of over/under fetching is solved by fetching resources as per the views present in the app. This makes the system tightly coupled.

## GraphQL

- The client can control all the data it needs from the server.
- In other words, the client doesn't get affected by under/over fetching or n+1 request problems — improves performance and reduces network bandwidth.
- In GraphQL, the request object is defined by the client.
- GraphQL simplifies data aggregation from multiple sources in a single API call.
- The resources are not exposed as per the views present in the app.
- GraphQL is an alternative to REST, not a replacement. Filtration logic can be implemented based on request parameters in REST as well.

## GraphQL Performance Issues

- The user can't run any query they want — the API must be well-designed using GraphQL domain language.
- Accessing multiple resources in a single API call might lead to deeply nested queries, potentially taking down a whole server.