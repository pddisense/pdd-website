---
layout: docs
title: Developing the dashboard
---

The PDD dashboard is a component used by analysts to configure campaigns and export results.
It is a lightweight server component serving a Web user interface.
The source code for the dashboard is available on GitHub: [https://github.com/pddisense/pdd-dashboard](https://github.com/pddisense/pdd-dashboard).

* TOC
{:toc}

## Build the server

The dashboard is written in Scala 2.12, and packaged as a Docker image to ease its deployment.
To build the dashboard, you will need a Java JDK 8 (Java 9 support is not complete in Scala 2.12, it has not been tested) and SBT (for Simple Build Tool).
The latter can be automatically downloaded and configured by using the `sbt` wrapper script included at the root of the repository.
The dashboard can be built with the following commands:

```bash
yarn install
yarn build
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

The test suite is also automatically launched by [Travis CI](https://travis-ci.com/pddisense/pdd-server) at each code push.
When everything succeeds, Travis is also configured to automatically publish the image on Docker Hub.

## Implementation notes

The server part is built on top of [Finatra](https://twitter.github.io/finatra/), which is a framework from Twitter helping with creating services.
This API mainly handles authentication, before forwarding calls to the API server.

The user interface is a single page application relying on this API to obtain its data.
It is written in Javascript ES6, using React and JSX, and is ultimately transpiled in plain Javascript by Babel.
The compilation process is managed by Webpack, which is call on `yarn build`.
The result of the compilation is directly written under `src/main/resources`, which will then be picked up and served by the Finatra server.
