---
layout: docs
title: Developing the server
---

The PDD server is the component with which the clients interact.
The source code for the server is available on GitHub: [https://github.com/pddisense/pdd-server](https://github.com/pddisense/pdd-server).

* TOC
{:toc}

## Build the server

The server is written in Scala 2.12, and packaged as a Docker image to ease its deployment.
To build the server, you will need a Java JDK 8 (Java 9 support is not complete in Scala 2.12, it has not been tested) and SBT (for Simple Build Tool).
The latter can be automatically downloaded and configured by using the `sbt` wrapper script included at the root of the repository.
The server can be built in one command:

```bash
./sbt compile
```

You can also create a local Docker image with the following command (requires Docker to be installed):

```bash
./sbt docker:publishLocal
```

## Running the tests

The tests are launched with SBT:

```bash
./sbt test
```

Note that in order to test the persistent storage, a MySQL instance is required to run all the tests.
By default, a MySQL server running on the same machine, listening on port 3306, and equipped with passwordless root user is expected, but you can override these settings with the environment variables `MYSQL_SERVER`, `MYSQL_USER` and `MYSQL_PASSWORD`.

The test suite is also automatically launched by [Travis CI](https://travis-ci.com/pddisense/pdd-server) at each code push.
When everything succeeds, Travis is also configured to automatically publish the image on Docker Hub.

## Implementation notes

The API server is built on top of [Finatra](https://twitter.github.io/finatra/), which is a framework from Twitter helping with creating services.
It provides two main REST APIs: a public one, used by the clients to implement [the cryptographic protocol](protocol.html), and a private one, used by the dashboard to configure the server and present the results to analysts.
The private API is authenticated via a Bearer token.
The API controllers are implemented in the [`ucl.pdd.server` package](https://github.com/pddisense/pdd-server/tree/master/src/main/scala/ucl/pdd/server).

The API is complemented by a set of background jobs, handling for example the creation of sketches and their aggregation.
The background jobs are implemented in the [`ucl.pdd.service` package](https://github.com/pddisense/pdd-server/tree/master/src/main/scala/ucl/pdd/service).

Various objects are manipulated by the server.
All those domain objects are implemented the [`ucl.pdd.domain` package](https://github.com/pddisense/pdd-server/tree/master/src/main/scala/ucl/pdd/domain).

  * **Campaigns** are a high-level concept materialising a list of queries of interest to be collected for research purposes.
For example, one could create a campaign monitoring flu-related queries such as "flu", "influenza", "fever", etc., while another researcher could create another campaign monitoring politics-related queries such as "Tory", "parliament" or "prime minister".
Of course, the PDD project is right now focused in the former.
Interestingly, a campaign also contains some parameters such as the size of groups within which to create users, which is a parameter having an impact on the utility of the results provided by PDD.
Multiple campaigns could be created with different group sizes, to examine the impact it has on the quality of results.
  * **Clients** represent the browser extensions installed by the users.
They provide minimal metadata such as the type of browser (e.g., "Chrome").
  * **Activity logs** are used to keep track of when clients ping the server.
This is notably used to compute optimal groups, e.g., by evicting clients that have been inactive for a long time.
  * **Sketches** are the encrypted vectors sent by the clients.
Sketches are first created empty, every night, and are expected to be filled by the clients when then come only the next day and ping the server.
They are short-lived objects, as they are garbage-collected if they stay empty (or cannot be aggregated) for too long (more than the delay configured in the campaign).
  * **Aggregations** are the decrypted vectors.
They are created every night, as soon as all sketches of a given group have been collected, and updated as new sketches arrive and allow to decrypt more groups.

The server has a pluggable storage layer, used to store all those objects.
Right now, in-memory storage and MySQL storage are provided.
The storage layer is implemented in the [`ucl.pdd.storage` package](https://github.com/pddisense/pdd-server/tree/master/src/main/scala/ucl/pdd/storage).
