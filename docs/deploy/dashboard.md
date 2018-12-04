---
layout: docs
title: Deploying the dashboard
---

The dashboard is an optional component providing an intuitive Web UI for administrators and analysts.
This page explains how to deploy the Web dashboard using Docker.

* TOC
{:toc}

## Deploying the dashboard

The API server is published as a Docker image on Docker Hub: [https://hub.docker.com/r/pddisense/pdd-dashboard/](https://hub.docker.com/r/pddisense/pdd-dashboard/).
You can retrieve it by running
```bash
docker pull pddisense/pdd-dashboard
```

The server accepts a lot of options, which you can visualise by running:
```bash
docker run pddisense/pdd-dashboard -help
```

## Configuring networking
By default, two ports are used:

  * Port 8001, which provides the main HTTP interface.
  * Port 9990, which provides [an HTTP administrative interface](https://twitter.github.io/twitter-server/Admin.html).

In the same manner than for the API server, both interfaces can be bound to another host/port by using the `-http.port=` and `-admin.port` flags.
Both take as argument a string formatted like `host:port`, where the host can possibly be left empty (but the colon still has to be included).
The administrative interface should *not* be publicly accessible and remain behind a firewall, while the main interface should be exposed on the Internet.

## Configuring security
The dashboard provides all its features by communicating with the API server.
The only required information is the `-api.access_token` information, which specifies the access token used to authenticate to the API server.
It should match the `-api.access_token` flag provided to the API server (or the value displayed on the standard output when the latter started).
The address of the API server can be specified with the `-api.server` flag, and if SSL is enabled you should use the `-api.ssl_hostname` to specify the matching hostname.
By default, it attempts to contact a local API server listening on port 8000.

By default the dashboard does not require authentication.
If the dashboard is publicly accessible, **it is highly recommended that you enable the authentication**.
This can be done by defining the `-master_password` flag to some secret value.
This password will be then used by the users to login interactively to the dashboard.
This value is by design different from the access token; the access token should not change very often, while the master password may possibly change as often as you need it to be.

Please note the essential difference between the access token and the master password.
The access token is intended as a means for internal authentication between services, while the master password is intended to be provided by users using the UI.
The goal to separate those (and not ask the users to provide the access token directly) is to allow to change the master password without having to do anything on the API server side.
Indeed, the master password is strictly internal to the dashboard, and can be changed easily by restarting only the dashboard.
You can even disable the password, e.g., if the dashboard is already deployed behind an authentication proxy.

## Example
As a reference, the following command is used to start the production dashboard:

```bash
docker run \
  --net host \
  --detach \
  --restart=always \
  --name pdd-dashboard \
  --env ENVIRONMENT=production \
  pddisense/pdd-dashboard \
    -api.access_token=<access token> \
    -api.server=localhost:8000 \
    -master_password=<master password> \
    -admin.port=:9001 \
    -http.port=:8001
```
