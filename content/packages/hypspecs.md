---
layout: default
title: Hypspec Files
permalink: hypspecs
nav_order: 7
parent: Packages Details
---

# Package specification files (Hypspec)
In order to control your packages state on agents via [HMC](hmc) you need to add for each package `Hypspec` file. 
> NOTE: you don't need it for local development and testing with `HDK`.

`Hypspec` is a json file which describes a package for HMC. It should be placed into the package directory and has a name in the following format: `<Package Name>-<Package Version>.hypspec`. File format is the following:
```json
{
  "name": "string",
  "ver": "X.X",
  "type": "[probe | esm | lib | scheduled]",
  "environment": {
    "os": [
      "windows"
    ],
    "os_ver": {
      "nt": [
        ">= 9600"
      ]
    },
    "engine": [
      "[service | sensor | all]"
    ],
    "agent_ver": [
      ">= 1.0"
    ]
  },
  "src": [
    "Source1.lua",
    "Source2.lua"
  ]
}
```
* `name` (**string**) - name of the probe, should be unique across all used packages repositories (this limitation will be fixed in future).
* `ver` (**string**) - string version of the package (e.g. `"0.1"`). 
* `type` (**string**) - [package type](packages#PackagesTypes).
* `environment.os` (**string**) - currently only `"windows"` is supported.
* `environment.os_ver.nt` (**array**) - supported build numbers of Windows.
* `environment.engine` (**string**) - [package engine](packages#PackagesEngines).
* `environment.agent_ver` (**string**) - version of agent where the package works.
* `src` (**array**) - list of package sources.

> NOTE: Packages versioning isn't supported yet. It is currently ignored but it's better to set `"0.1"` for all your packages. 