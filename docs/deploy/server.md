---
layout: docs
title: Deploying the server
---

Private Data Donor is made of three different components: an API server, a Web dashboard and a Chrome extension.
The API server is a mandatory component with which the clients communicate.
This page explains how to deploy the API server using Docker.
If you want to get a deeper understanding of the architecture, you can read [the dedicated page](../contribute/architecture.html) in the developer guide.

* TOC
{:toc}

## Installing Docker

The PDD platform is packaged as a Docker image to ease its installation on heterogeneous platforms.
Consequently, you can install PDD on any platform supported by Docker, i.e., Linux, Mac OS and Windows.
First of all, please follow [Docker's documentation](https://docs.docker.com/install/) to install the Docker Engine on the target machine.

Please check that Docker is correctly installed by running `docker version`.    

## Deploying the API server

The API server is published as a Docker image on Docker Hub: [https://hub.docker.com/r/pddisense/pdd-server/](https://hub.docker.com/r/pddisense/pdd-server/).
You can retrieve it by running
```bash
docker pull pddisense/pdd-server
```

The server accepts a lot of options, which you can visualise by running:
```bash
docker run pddisense/pdd-server -help
```

## Configuring networking

By default, two ports are used:

  * Port 8000, which provides the main HTTP interface.
  * Port 9990, which provides [an HTTP administrative interface](https://twitter.github.io/twitter-server/Admin.html).

Both interfaces can be bound to another host/port by using the `-http.port=` and `-admin.port` flags.
Both take as argument a string formatted like `host:port`, where the host can possibly be left empty (but the colon still has to be included).
The administrative interface should *not* be publicly accessible and remain behind a firewall, while the main interface should be exposed on the Internet.

## Configuring storage
By default, the server uses an in-memory storage, which is by definition not persistent.
While this may be useful for local testing, a production setup requires to use a proper persistent storage.
For now, the only implementation is a MySQL storage, which is enabled by specifying the server address with tge `-mysql_server` flag, e.g., `-mysql_server=localhost:3306`.
Then, the `-mysql_user`, `-mysql_password` and `-mysql_database` flags can be used to override, respectively,
the MySQL username, password and database name.
By default, it connects to a database named `pdd` as the `root` user and no password.

## Configuring security
The private endpoints are secured by the means of an access token, specified with the `-api.access_token` flag.
By default, a random access token is generated and printed in the standard output.
If you want to connect the dashboard to the API server later on, you may wish to generate your own access token and provide it a an option.

## Example
As a reference, the following command is used to start the production API server:

```bash
docker run \
  --net host \
  --detach \
  --restart=always \
  --env 'SENTRY_DSN=https://<public>:<private>@sentry.io/302347' \
  --env ENVIRONMENT=production \
  --name pdd-server \
  pddisense/pdd-server \
    -mysql_server=localhost:3306 \
    -mysql_user=pdd \
    -mysql_password=<mysql password> \
    -datadog_server=127.0.0.1:8125 \
    -geocoder=maxmind \
    -api.access_token=<access token> \
    -http.port=:8000 \
    -admin.port=:9000
```
