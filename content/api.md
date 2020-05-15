---
layout: default
title: Development Libraries
nav_order: 6
permalink: api
---

# Development Libraries

To simplify and speed up packages development developers can use two libraries. 
## hplib
<a href="/runtime/modules/hp.html" target="_blank">Documentation</a>

The library is a part of Hyperionix Agent and contains the following:
* Some agent information so it could be accessed from the packages.
* Entities API - functions to get deep information about some system objects in one call.
* Miscellaneous modules and some third party libraries.

## sysapi
* <a href="/sysapi/index.html" target="_blank">Documentation</a>
* <a href="https://github.com/hyperionix/sysapi" target="_blank">Source</a>

The library implements some classes and modules to simplify working with WinAPI and NT API. With this library developers can work
with OS objects like files, processes, registry etc. Actually the library isn't part of the Hyperionix project and could be used independently but it is being actively expanded and improved while maintaining compatibility with the Hyperionix project. Feel free to create PR with your changes to the library and we will add it.


