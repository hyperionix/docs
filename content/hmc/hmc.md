---
layout: default
title: Hyperionix Management Console
permalink: hmc
has_children: true
nav_order: 5
---

# Hyperionix Management Console
Hyperionix Management Console, or HMC for short, lets you control multiple agents from a central location. You can use
it to deploy new agents, control existing agents, add or remove probes from agents, send commands to agents and more.

To access HMC go to [https://admin.hyperionix.com](https://admin.hyperionix.com/).

## Sign-up

An account is required to use HMC. Your agents will be tied to your specific account and no one else will be able to
access or view them.  If you don't yet have an account, click on [Sign up](https://admin.hyperionix.com/#!/auth/signup)
at the bottom of the page.

![Sign-up](assets/img/signup.png)

## Download Agent and HDK

The first thing you want to do after signing-up is download the agent installer so you can deploy it on your machines,
or download the [HDK](hdk) so you can start building your own hooks and probes. To download the tools, use the Download
icon on the left menu.

![Download](assets/img/download.png)

When installing agents, you will be asked for a license key. To get your key, go to the Account menu on the top right
and choose Keys. From there, click the Copy button to copy your license key to the clipboard.

![Keys](assets/img/key.png)

You will be asked for the key during installation. Simply paste the key you got from HMC into the installer.

![Agent Installer](assets/img/key-install.png)

## Enable Agents

Once an agent is installed and connected, you will see it in the Agents page. You can access the Agents page from the
left menu. Agents are always disabled when they first connect. This allows you greater control of what agents to allow
in your network. Disabled agents are highlighted with a red background to make them easier to notice.

![Disabled agent](assets/img/agent-disabled.png)

To enable an agent:
1. Select the agent by clicking it
2. Click the Actions menu on the top left
3. Select Enable to enable the agent

![Disabled agent](assets/img/agent-enable.png)

You can also select multiple agents at once and enable them in bulk.

## Agent Probe Management
Once the agents are installed, connected and enabled; it's time to install and configure some probes. With probse your
agents can start tracking or modifying the system behavior. 
### Installing Probes
1. Go to _Agents_ page.
2. Select agents where probe should be installed.
3. Choose _Probes_ tab in the bottom half of the page.
4. Click on _Add probes_ button, choose probes you want to install, and finally click _Add probes_
5. Click _Apply configuration_. Wait until the operation will be completed.
### Removing Probes
1. Choose an agent.
2. Go to _Probes_ tab 
3. Choose probes you want to remove
4. _Actions -> Uninstall probes_

## Package Repositories
To use probes from repositories other than the built-in Hyperionix library of probes, you can add your own repositories
or even third party repositories. To manage repositories use the Repositories page accessible through the left menu.

Our main [Hyperionix repository](https://github.com/hyperionix/packages) is added by default, but can be removed.

![Repositories](assets/img/repos.png)

### Adding New Repositories
> NOTE: Only GitHub and BitBucket repositories are currently supported.

Go to _Actions -> Add_ menu. To add a repository you need to set its name and address like for `git clone` command. Both HTTPS and SSH are supported. You can also choose a branch you want to use.
* For GitHub private repositories you have to create a [personal access token](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line) with **repo** access and add it to _API token_ field.
* For BitBucket private repositories you have to add the Hyperionix access key to your repository settings.
    1. On _Repositories_ page copy the key using button _Copy public key_
    2. Add it to your repository settings using [this](https://confluence.atlassian.com/bitbucket/access-keys-294486051.html) documentation starting from Step 3.

### Repository Synchronization
HMC doesn't automatically pull your repository. If you push changes to your repository, you must tell HMC to pull them
manually or they will not be used. You must also manually synchornize a repository after adding it for the first time.

1. Select the repositories you want to pull
2. _Actions -> Synchronize_
