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
