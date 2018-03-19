# Octokit routes

> machine-readable, always up-to-date GitHub REST API route specifications

[![Build Status](https://travis-ci.org/octokit/routes.svg?branch=master)](https://travis-ci.org/octokit/routes) [![Greenkeeper badge](https://badges.greenkeeper.io/octokit/routes.svg)](https://greenkeeper.io/)

## Downloads

- All routes: [octokit.github.io/routes/index.json](https://octokit.github.io/routes/index.json)
- All routes for a scope, e.g. `repos`: [octokit.github.io/routes/routes/repos.json](https://octokit.github.io/routes/routes/repos.json)
- A single route, e.g. `GET /repos/:owner/:repo`: [octokit.github.io/routes/routes/repos/get.json](https://octokit.github.io/routes/routes/repos/get.json)

## Example

Example route definition

```
{
  "name": "Lock an issue",
  "enabledForApps": true,
  "method": "PUT",
  "path": "/repos/:owner/:repo/issues/:number/lock",
  "params": [
    {
      "name": "lock_reason",
      "type": "string",
      "description": "The reason for locking the issue or pull request conversation. Lock will fail if you don't use one of these reasons:  \n\\* `off-topic`  \n\\* `too heated`  \n\\* `resolved`  \n\\* `spam`",
      "required": false
    }
  ],
  "description": "Users with push access can lock an issue or pull request's conversation.\n\nNote that, if you choose not to pass any parameters, you'll need to set `Content-Length` to zero when calling out to this endpoint. For more information, see \"[HTTP verbs](/v3/#http-verbs).\"",
  "documentationUrl": "https://developer.github.com/v3/issues/#lock-an-issue"
}
```

## Usage as Node module

```
const ROUTES = require('@octokit/routes')
```

returns an object with keys being the route scopes such as `activity`, `issues`,
`repositories`, etc (one for navigation header in the sidebar at https://developer.github.com/v3/).

The value for each scope is an array of REST API endpoint specification.

If you don’t need all routes definitions, you can require a scope or a specific
route definition instead

```js
const REPO_ROUTES = require('@octokit/routes/routes/repos')
const GET_REPO_ROUTE = require('@octokit/routes/routes/repos/get')
```

## How it works

This package updates itself using a daily cronjob running on Travis. All
[`routes/*.json`](routes/) files as well as [`index.js`](index.js) is
generated by [`bin/octokit-rest-routes.js`](bin/octokit-rest-routes.js). Run
 `node bin/octokit-rest-routes.js --usage` for instructions.

The update script is scraping [GitHub’s REST API](https://developer.github.com/v3/)
documentation pages and extracts the meta information using [cheerio](https://www.npmjs.com/package/cheerio)
and a ton of regular expressions :)

For simpler local testing and tracking of changes all loaded pages are cached
in the [`cache/`](cache/) folder.

### 1. Find documentation pages

- Index page cached in [`cache/v3/index.html`](cache/v3/index.html)
- Result cached in [`cache/pages.json`](cache/pages.json)

Opens https://developer.github.com/v3/, find all documentation page URLs
in the side bar navigation.

### 2. On each documentation page, finds sections

- Documentation pages cached in `cache/v3/*/index.html`, e.g. [`cache/v3/repos/index.html`](cache/v3/repos/index.html)
- Documentation sub pages cached in `cache/v3/*/*/index.html`, e.g. [`cache/v3/repos/branches/index.html`](cache/v3/repos/branches/index.html)
- All sections found on pages are cached next to the `index.html` files, e.g. [`cache/v3/repos/sections.json`](cache/v3/repos/sections.json)
- HTML of sections are stored next to the `index.html` files, e.g. [`cache/v3/repos/create.html`](cache/v3/repos/create.html)

Loads HTML of each documentation page, finds sections in page.

### 3. In each section, finds endpoints

- Some sections don’t have endpoints, such as [Notifications Reasons](https://developer.github.com/v3/activity/notifications/#notification-reasons)
- Some sections have multiple endpoints, see [#3](https://github.com/octokit/routes/issues/3)

Loads HTML of documentation page section. Turns it into [`routes/*.json`](routes/) file.
In some cases the HTML cannot be turned into an endpoint using the implemented patterns.
For these cases [custom overrides](lib/endpoint/overrides) are defined.

## LICENSE

[MIT](LICENSE.md)
