## test-electron-builder

A stripped-down example that shows an oddity in the electron packaging logic with `yarn` workspaces:

# Steps to reproduce:
 - Install the dependencies:
```
yarn
```
 - Open the example application and check the `vscode-uri` version at runtime. Due to the saved `yarn.lock`, you should see `1.0.8`.

 ```
 yarn --cwd ./examples/app start
 ```
 - Package the electron application with `electron-builder` and run it. You will see, the `vscode-uri` version is `2.1.1`.
```
yarn --cwd ./examples/app package
```

# Oddity:
Executing `yarn why vscode-uri` shows, the dependency hoisting is correct.
```
yarn why vscode-uri
yarn why v1.15.2
[1/4] 🤔  Why do we have the module "vscode-uri"...?
[2/4] 🚚  Initialising dependency graph...
[3/4] 🔍  Finding dependency...
[4/4] 🚡  Calculating file sizes...
=> Found "vscode-uri@1.0.8"
info Has been hoisted to "vscode-uri"
info Reasons this module exists
   - "workspace-aggregator-1922fc2c-60dc-48b8-98d2-d4fccb2d0f6d" depends on it
   - Hoisted from "_project_#package-a#vscode-uri"
   - Hoisted from "_project_#package-b#vscode-uri"
info Disk size without dependencies: "104KB"
info Disk size with unique dependencies: "104KB"
info Disk size with transitive dependencies: "104KB"
info Number of shared dependencies: 0
=> Found "vscode-json-languageserver#vscode-uri@2.1.1"
info This module exists because "_project_#package-c#vscode-json-languageserver" depends on it.
info Disk size without dependencies: "104KB"
info Disk size with unique dependencies: "104KB"
info Disk size with transitive dependencies: "104KB"
info Number of shared dependencies: 0
=> Found "vscode-json-languageservice#vscode-uri@2.1.1"
info This module exists because "_project_#package-c#vscode-json-languageserver#vscode-json-languageservice" depends on it.
info Disk size without dependencies: "104KB"
info Disk size with unique dependencies: "104KB"
info Disk size with transitive dependencies: "104KB"
info Number of shared dependencies: 0
✨  Done in 0.34s.
```

 - The older version (`1.0.8`) of `vscode-uri` was correctly hoisted to the top level `node_modules` folder as both `package-a` and `package-b` has a hard dependency on it. Check the `version` in `./node_modules/vscode-uri/package.json`.
 - The more recent version (`2.1.1`) of `vscode-uri` was correctly not hoisted. Check `version` in `node_modules/vscode-json-languageserver/node_modules/vscode-uri/package.json`.
 - The `electron-builder` does not seem to respect [`semver`](https://semver.org) and the `yarn` workspace, and pulls in the incorrect (`2.1.1.`) version of `vscode-uri` into the bundled electron application.