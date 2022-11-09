# Malloy Composer Demo

The Malloy Composer Demo is provided as a working example of an application built on top of Malloy. If you have any questions about getting it running, please reach out to us for help! If you find bugs or have feature requests, you can submit them as issues in this repo. [Learn how to use the composer](https://docs.google.com/presentation/d/18KUl_rrz2K-hbsiKJYS3rtTcYxZMXKklyPllLmTtIYY/edit#slide=id.g1269816dcbe_0_140)

The composer is only intended for demo purposes, and is not a finished or supported product.

_GitHub mutes videos by default, so make sure to unmute._

https://user-images.githubusercontent.com/7178946/170373869-3cf43dd2-25c4-4ed0-b038-450c33903ad5.mov

## Running the Composer

1. `npm install` to install package dependencies
2. `git submodule init` and
3. `git submodule update` to install git dependencies

Make sure you have a [database connected](https://looker-open-source.github.io/malloy/documentation/connection_instructions.html), and you'll also likely want to set up the [VS Code Extension](https://github.com/looker-open-source/malloy#installing-the-extension) to view and edit Malloy files.

### Launch the Composer

Run:

1. `npm run build` (you need to do this in addition to the above build in the top-level directory)
2. `npm run start`

This will start a desktop application. You should see any sources defined in `.malloy` files you place in a the Malloy models directory (by default, this is the samples directory but can be configured, as described below) listed in the "Select analysis..." menu at the top left. If you don't already have Malloy models built you'd like to work with, try making a copy of one of the [samples](https://github.com/malloydata/malloy-samples); these are all built on public BigQuery datasets!

Troubleshooting notes:

- If you have models in your Malloy models directory and one or all of them are not showing up in the composer menu, you may have an error in your Malloy code. Try opening them up in VS Code with the Malloy Extension installed to find the problem.
- You'll need to define a [source](https://looker-open-source.github.io/malloy/documentation/language/source.html) for it to be explorable; top-level named queries that are not inside a source are not explorable.

### Set up Query Saving

The composer can write saved queries back to `.a.malloy` files in the Malloy models directory (see below).

1. Create a new file with the suffix `.a.malloy` (e.g. `flights.a.malloy`). You'll need separate ones for each source you want to make explorable.
2. [Import](https://looker-open-source.github.io/malloy/documentation/language/imports.html) the base file in this `.a.malloy` file, then create a refinement of a source named in the base file. For example, if your base file looks like:

```malloy
source: flights_base is table('malloy-data.faa.flights'){}
```

Your `.a.malloy` file might look like this:

```
import "file:///Users/anikaks/malloy/flights.malloy"

source: flights is flights_base {}
```

You should now see the name of your new source appear in the top left menu, and when you click the start icon in the top menu you should be able to save named queries and see them appear inside the new source. _Note: Only the **last source** in a `.a.malloy` file will appear in the menu._

The composer is a two-way tool; saved queries are saved into this source by the app, but you can also add/edit named queries, add fields, joins, etc.

### Settings

The Malloy models directory is `./malloy-samples` by default, but you can set it to another directory by adding a `composer_config.json` file (see `composer_config.sample.json` for a sample of this file).

For example, a `composer_config.json` with the following content would configure the Composer to look for models in the `~/malloy` directory.

```
{
  "modelsPath": "~/malloy"
}
```

### Packaging Notes

- Download XCode
- Join developer program
- Download Developer ID — G2 (from https://www.apple.com/certificateauthority/)
- Make a cert signing request
- Get a certificate from Apple (using cert signing request) — Make sure it’s a "Developer ID Application”
- Create an `env` file:

```
export NOTARIZER_APPLE_ID=<your apple ID>
export NOTARIZER_APPLE_ID_PASSWORD=<your apple id app password>
export SIGNER_IDENTITY=<name of signing certificate>
```

- `source env`
- `npm run package` might take ~10m to run while Apple notarizes the build
- If you see the following (but the command keeps running), just wait it out.

```
WARNING: Code sign failed; please retry manually. Error: Command failed: spctl --assess --type execute --verbose --ignore-cache --no-cache /var/folders/6q/dq6hklkn309f1_7737gmbvvw00r7jt/T/electron-packager/darwin-x64/malloy-composer-demo-darwin-x64/malloy-composer-demo.app
/var/folders/6q/dq6hklkn309f1_7737gmbvvw00r7jt/T/electron-packager/darwin-x64/malloy-composer-demo-darwin-x64/malloy-composer-demo.app: rejected
source=Unnotarized Developer ID
```

## Running against a local version of Malloy

1. In your local Malloy repository, run `npm link -ws`. That will make your development packages locally available for development.
2. In your VS Code extension repository, run `npm run malloy-link` to use your local Malloy packages.
3. If you make changes to Malloy that are required by the extension, merges those into main, and that will trigger an automatic developer release of Malloy.
4. Once that release completes, run `npm run malloy-update` to update dependencies to that release. This will break the link to your local version of Malloy, so if you want to resume local development, re-run `npm run malloy-link`
5. To manually unlink without updating, you may run `npm run malloy-unlink`

### Debugging

## Using VSCode

- From VSCode Run & Debug Panel Select "Launch Composer" from the dropdown, then "Start Debugging" using the Run button or `F5`
- Then select "Launch Composer" from the dropdown, then "Start Debugging" using the Run button or `F5`
- To connect to the render process select "Attach to Composer Render Process"

## DuckDB WASM Mode

There is a `duckdb-wasm` folder which should be served on a webserver in order to run an instance of the DuckDB WASM Mode Composer. The `duckdb-wasm/dist` folder contains the compiled app after running `npm run build-duckdb-wasm`. You can also run `npm run start-duckdb-wasm` to run the DuckDB WASM Mode Composer on port 9999.

The WASM Mode Composer can be easily added to any project that already uses the Fiddle, or it can be added to any repo with Malloy files using CSV or parquet files to simply serve the Composer experience on top of those files. All you need is to host the generated `app.js` and `app.css` files on some CDN (it can be the same CDN that serves the data and malloy files, or a different one), then to add a `composer.json` file that lists the available datasets, e.g.

```json
[
  {
    "name": "Airports",
    "dataTables": ["airports.parquet"],
    "modelPath": "./airports.malloy",
    "readme": "./airports.md"
  }
]
```

Finally, add an `index.html` file, like this:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>Malloy Composer</title>
    <link rel="stylesheet" href="dist/app.css" />
    <script src="dist/app.js" async defer></script>
  </head>
  <body>
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <div id="root"></div>
  </body>
</html>
```