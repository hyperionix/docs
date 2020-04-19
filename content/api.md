---
layout: default
title: API Docs
nav_order: 6
permalink: api
---

# API Docs

To simplify and speed up packages developemt devleopers can use two libraries. 
## hplib
<a href="/runtime/modules/hp.html" target="_blank">Documentation</a>

The library is a part of Hyperionix Agent and contains the following:
* Some agent information so it could be accessed from the packages.
* Entities API - functions to get deep information some system objects in one call.
* Miscellaneous modules and some thirdparty libaries.

## sysapi
* <a href="/sysapi/index.html" target="_blank">Documentation</a>
* <a href="https://github.com/hyperionix/sysapi" target="_blank">Source</a>

The library implements some classes and modules to simplify working with WinAPI and NT API. With this library developers can work
with OS objects like files, processes, regsitry etc. Actualy the library isn't part of Hyperionix project and could be used independently but it is constantly expand and improved with keeping Hyperionix in mind. Feel free to create PR with your changes to the library and we will add it.
