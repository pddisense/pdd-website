---
layout: docs
title: Developing the client
---

The PDD client is provided as a browser extension, that is to be installed in the users' browsers.
The source code for the client is available on GitHub: [https://github.com/pddisense/pdd-client](https://github.com/pddisense/pdd-client).

* TOC
{:toc}

## Build the extension

Chrome extensions are written in Javascript, and packaged as ZIP files before being uploaded on the Chrome Web Store.
We provide here an overview of the inner workings of the PDD extension, more information about developing Chrome extensions can be found [in the official documentation](https://developer.chrome.com/extensions/devguide).
To build the extension, you will need [Node ≥10.9.0](https://nodejs.org) and [Yarn](https://yarnpkg.com).

```bash
yarn install
yarn build
```

The Chrome extension is made of two main parts: a background script, which implements [the cryptographic protocol](protocol.html), and an options page, providing the user with some controls over the data collection process.
It is written in Javascript ES6, transpiled with Babel, and the options page uses React for the user interface.

## Running the tests

The tests are launched with Yarn:
```bash
yarn test
```

Please allow some time for the tests to complete, as the cryptographic operations are quite extensive.

## Implementation notes

The Chrome extension uses the [local storage API](https://developer.chrome.com/extensions/storage) (which is different from HTML5 local storage) to store the user preferences as well as the cryptographic keys.
The [history API](https://developer.chrome.com/extensions/history) is used to access the browser's history.
This solution was preferred over monitoring every browsed URL on-the-fly and storing the ones corresponding to Web searches in a local storage.
Integrating directly with the history API requires a lower-level permission.
More importantly, it also allows to remain consistent with the user's wishes if he deletes some items from the browsing history, which we consider as an indication he may not feel comfortable with sharing that with us either.
For now, only Google searches are supported, but it would require minimal effort to support other search engines.
Finally, the [alarms API](https://developer.chrome.com/extensions/alarms) is used to schedule the pings.

## Publish the Chrome extension

There is a release script, which is a small helper to create a packaged extension, ready to be uploaded on the Chrome Web store.
```bash
./bin/release
```

This will create a `dist/chrome.zip` package.
You then need to upload the latest package via the [Chrome Developer Dashboard](https://chrome.google.com/webstore/developer/dashboard).

## Public API specification

The client is expected to interact with the API server through a set of public endpoints, which are used to implement [the cryptographic protocol](protocol.html).
They are considered public in the sense that any client can use it, and there is no explicit authentication.
Instead, the client's unique name should be considered as a private information (and thus not to be shared), used to identify itself with the server.

### POST /api/clients

This endpoint is used to create a new client.
The request body should contain the following attributes:

| Request attribute                   | Description |
|-------------------------------------|-------------|
| `publicKey`<br>string               | Public key. |
| `browser`<br>string                 | Browser identifier (e.g., 'chrome', 'firefox'). |
| `externalName`<br>string (optional) | External name voluntarily specified by the user. |
{: class="table table-api" }

If the request is successful, a 200/OK response will be returned, with the following attributes:

| Response attribute                  | Description |
|-------------------------------------|-------------|
| `name`<br>string                    | Client's unique identifier. |
| `createTime`<br>string              | Time at which this client was created (ISO 8601). |
| `publicKey`<br>string               | Public key. |
| `browser`<br>string                 | Browser identifier (e.g., 'chrome', 'firefox'). |
| `externalName`<br>string (optional) | External name voluntarily specified by the user. |
{: class="table table-api" }

If the request contains an invalid field, a 400/Bad Request response will be returned.
If for any reason a client's identifier could not be generated, a 409/Conflict response will be returned (and the request can be safely retried).

### PATCH /api/clients/:name

This endpoint is used to modify the attributes of an existing client.
The request body should contain the following attributes:

| Request attribute                   | Description |
|-------------------------------------|-------------|
| `externalName`<br>string (optional) | External name voluntarily specified by the user. |
{: class="table table-api" }

If the request is successful, the response body will contain the same attributes than when creating a client.
If the specified client does not exist, a 404/Not Found response will be returned.
If the request contains an invalid field, a 400/Bad Request response will be returned.

### DELETE /api/clients/:name

This endpoint is used to delete an existing client.
If the request is successful, an empty 200/OK response will be returned.
If the specified client does not exist, a 404/Not Found response will be returned.

### GET /api/clients/:name/ping

This endpoint is used by a client to get the list of sketches waiting to be filled.
If the request is successful, a 200/OK response will be returned, with the following attributes:

| Response attribute                                          | Description               |
|-------------------------------------------------------------|---------------------------|
| `submit`<br>object[]                 | List of sketches to fill. |
| `submit[].sketchName`<br>string      | Name of the sketch to submit |
| `submit[].startTime`<br>string       | Start time of collection period (ISO 8601). |
| `submit[].endTime`<br>string         | End time of collection period (ISO 8601). |
| `submit[].vocabulary`<br>object      | List of the queries of interest for this sketch. The whole vocabulary is sent every time. |
| `submit[].vocabulary.queries`<br>object[]                    | |
| `submit[].vocabulary.queries[].exact`<br>string (optional)   | If this is defined, this is an "exact" query, meaning the search has to match exactly the query. |
| `submit[].vocabulary.queries[].terms`<br>string[] (optional) | If this is defined, this is a "terms" query, meaning the search has to contain all the provided terms (order does not matter). |
| `submit[].publicKeys`<br>string[]    | List of the other clients' public keys. It includes the key of the client hitting the endpoint. |
| `submit[].round`<br>number           | Round number. In practice, it corresponds to the index of the day since the campaign started. |
| `vocabulary`<br>object[]             | List of all queries of interest across all campaigns. This is not intended to be used when computing the sketches, but may be stored locally by the clients and later exposed to the users for transparency purposes. This has the same structure than `submit[].vocabulary`. |
| `nextPingTime`<br>string             | Next time at which the client should ping the server. This is only a hint, and should in practice correspond to once a day. |
{: class="table table-api" }

If the specified client does not exist, a 404/Not Found response will be returned.

### POST /api/clients/:name/ping

This endpoint is used by a client to get the list of sketches waiting to be filled.
This is a variant of the same path with a GET method, allowing the client to specify some metadata.
The request body should contain the following attributes:

| Request attribute             | Description |
|-------------------------------|-------------|
| `extensionVersion`<br>string | Current version of the installed extension. |
| `timezone`<br>string | Timezone the client is currently located in (e.g., 'Europe/London'). |
{: class="table table-api" }

The same responses than for its GET counterpart can be returned.

### PATCH /api/sketches/:name

This endpoint is used by a client to fill the values of a sketch.
The request body should contain the following attributes:

| Request attribute             | Description |
|-------------------------------|-------------|
| `encryptedValues`<br>string[] | List of encrypted values. There should be one item per query in the sketch's associated vocabulary. Values are represented as strings (and not numbers) because they will likely exceed the maximum integer size. Their decimal string representation should thus be used. |
{: class="table table-api" }

If the request is successful, an empty 200/OK response will be returned.
If the specified sketch does not exist, a 404/Not Found response will be returned.
