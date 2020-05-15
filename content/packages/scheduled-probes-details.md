---
layout: default
title: Scheduled Probes
nav_order: 3
parent: Packages Details
permalink: scheduled-probes
---

# Scheduled Probe package
This kind of package is used to perform a job periodically inside Hyperionix Service process. The most typical use case is to collect system data like process list, services list etc. The probe declaration looks like this:
```lua
ScheduledProbe {
  name = "string",
  interval = integer,
  callback = function()
    -- ...
  end
}
```
* `name` (**string**) - name of the probe, should be unique across all used packages repositories (this limitation will be fixed in future).
* `interval` (**integer**) - period of `callback` execution in milliseconds.
* `callback` (**function**) - callback to be executed.

Check [this](https://github.com/hyperionix/packages/blob/master/scheduled-probes/Services%20List/Services%20List.lua) probe which is intended to get list of user-mode services.