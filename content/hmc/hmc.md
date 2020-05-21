---
layout: default
title: Hyperionix Management Console
permalink: hmc
has_children: true
nav_order: 4
---

# Hyperionix Management Console
[Login](https://admin.hyperionix.com/#!/auth/login) on the HMC.

## Agent downloading and installation
1. In the left side menu go to _Download_ and download Agent installer named _Hyperionix-%version%.exe_
2. Click to the Account menu in the right up corner and choose _Keys_ menu. Agent needs the key to pass authentication on HMC.
3. Run Agent installer on a target system and set the key during installation in _License Key_ field. Also you can optionally set your Splunk server address and choose a name for your agent.

## Enable an agent
After agent installation go to the _Agent_ menu. Once the agent has been installed you need to acknowledge this installation. By default all new agents are disabled and they are highlighted with red color. 
1. Choose agents you want to enable
2. Enable them with _Actions_ menu: _Actions -> Enable_

## Package repositories
In the left side menu there is _Repositories_ menu where you can manage your package repositories and browse available packages. By default you have our main [Hyperionix repository](https://github.com/hyperionix/packages) added. If you are not gonna add one more you can follow to [installing probes](#agent-probe-state-management)

### Adding a new repository
> NOTE: Only github and bitbucket repositories are currently supported.

Go to _Actions->Add_ menu. To add a repository you need to set its name and address like for `git clone` command. Both HTTPS and SSH are supported. You can also choose a branch you want to synchronize.
* For github private repositories you have to create [personal access token](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line) with **repo** access and add it to _API token_ field.
* For bitbucket private repositories you have to add the Hyperionix access key to your repository settings.
    1. On _Repositories_ page copy the key using button _Copy public key_
    2. Add it to your repository settings using [this](https://confluence.atlassian.com/bitbucket/access-keys-294486051.html) documentation starting from Step 3.

### Repository synchronization
After adding a new repository of if you want to fetch repository updates you should do the following:
1. Choose repositories you want to update
2. _Actions -> Synchronize_

## Agent probe state management
After you install agents and add necessary repositories you can install, configure and remove probes on your agents. 
### Probes installation
1. Go to _Agents_ page.
2. Choose agents you want to install probes.
3. Choose _Probes_ tab in the bottom of the page.
4. Click on _Add probes_ button and choose probes you want to install and click _Add N probes_
5. Click _Apply configuration_. Wait until the operation will be completed.
### Probes removing
1. Choose an agent.
2. Go to _Probes_ tab 
3. Choose probes you want to remove
4. _Actions -> Uninstall probes_


