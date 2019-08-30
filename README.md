# backfill

A JavaScript caching library for reducing build time.

**🔌 Easy to install**: Wrap your build commands inside
`backfill -- [command]`  
**☁️ Remote cache**: Store your cache on Azure Blob or as an npm package  
**⚙️ Fully configurable**: Smart defaults with cross-package and per-package
configuration and environment variable overrides

_backfill is under active development and should not be used in production,
yet._

## Why

When you're working in a multi-package repo you don't want to re-build packages
that haven't changed. By wrapping your build scripts inside `backfill` you
enable storing and fetching of build output to and from a local or remote cache.

Backfill is based on two concepts:

1. **Hashing**: It will hash the files of a package, its dependencies and the
   build command
2. **Caching**: Using the hash key, it will look for build output from a local
   or remote cache. If there's a match, it will backfill the package using the
   cache. Otherwise, it will run the build command and persist the output to the
   cache.

## Install

Install backfill using yarn:

```
$ yarn add --dev backfill
```

Or npm:

```
$ npm install --save-dev backfill
```

## Usage

```
backfill -- [command]
```

Typically you would wrap your npm scripts inside `backfill`, like this:

```json
{
  "name": "package",
  "scripts": {
    "build": "backfill -- tsc"
  }
}
```

**Note**: Backfill will only generate a hash for the package it was called from.
In order to accurately create a hash representation of that package's
dependencies, backfill assumes that those packages have already been built using
backfill. Using tools like Lerna, wsrun or Rush, you can build packages in a
topologically sorted order such that this assumption is fulfilled.

### `--hash-only`

Sometimes you can't enable backfill for all packages. For these packages, simply
add `backfill --hash-only` to the build commands to only generate a hash of that
package without caching anything. Creating a hash of all packages in your repo
is important as dependent packages using backfill will look for hash keys from
their dependencies.

### `--audit`

Backfill can only bring back build output from the folders it was asked to
cache. A package that modifies or adds files outside of the cached folders will
not be brought back to the same state as when it was initially built when used
together with backfill. To help you debug this you can add `--audit` to your
`backfill` command. It will listen to all file changes in your repo (it assumes
you're in a git repo) will running the build command and then report on all
files that got changed that are outside of the cache scopes.

### Configuration

Backfill will look for `backfill.config.js` recursively from the package it is
called from and combine those configs together.

To configure backfill, simply export a config object with the properties you
wish to override:

```js
module.exports = {
  cacheStorageConfig: {
    provider: "azure-blob",
    options: { ... }
  }
};
```

The default configuration object is:

```js
{
  packageRoot: "path/to/package",
  name: "name-of-package",
  cacheStorageConfig: { provider: "local" },
  folderToCache: "lib",
  useTelemetry: true,
  localCacheFolder: "node_modules/.cache/backfill",
  hashFileFolder: "node_modules/.cache/backfill",
  telemetryFileFolder: "node_modules/.cache/backfill",
  watchGlobs: {
    folders: { exclude: ["lib", "node_modules"], include: ["*"] },
    files: { include: ["*"] }
  }
}

```

The configuration type is:

```ts
export type Config = {
  name: string;
  packageRoot: string;
  cacheStorageConfig: CacheStorageConfig;
  folderToCache: string;
  useTelemetry: boolean;
  localCacheFolder: string;
  hashFileFolder: string;
  telemetryFileFolder: string;
  telemetryReportName?: string;
  watchGlobs: WatchGlobs;
};
```

#### Environment variable

You can override configuration with environment variables. Backfill will also
look for a `.env`-file in the root of your repository, and load those into the
environment. This is useful when you don't want to commit keys and secrets to
your remote cache, or if you want to commit a read-only access key in the repo
and override with a write and read access key in merge build.

See `getEnvConfig()` in `./packages/backfill/src/config.ts`.

## Set up remote cache

### Microsoft Azure Blog Storage

[TODO]

### Npm package

[TODO]

## Telemetry

Some telemetry is gathered and stored to disk. Nothing gets uploaded. It is
simply stored on disk for your own analysis. You can turn if off by putting
`useTelemetry: false` in `backfill.config.js`.

## Contributing

This project welcomes contributions and suggestions. Most contributions require
you to agree to a Contributor License Agreement (CLA) declaring that you have
the right to, and actually do, grant us the rights to use your contribution. For
details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether
you need to provide a CLA and decorate the PR appropriately (e.g., status check,
comment). Simply follow the instructions provided by the bot. You will only need
to do this once across all repos using our CLA.

This project has adopted the
[Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the
[Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any
additional questions or comments.