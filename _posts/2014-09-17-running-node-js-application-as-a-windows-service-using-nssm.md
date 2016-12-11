---
layout: post
title: Running Node.js application as a Windows Service using NSSM
date: 2014-09-17 14:13
author: arenhage
comments: true
categories: [Home, node.js, nodejs, nssm, service, windows, windows service]
---
Came across an easy way of deploying your Node.js application as a Windows Service using NSSM (Non-Sucking Service Manager)

<!--more-->

To do the following you will need to running the command-prompt as Administrator.

NSSM is available for download at:
<a title="http://nssm.cc/" href="http://nssm.cc/">http://nssm.cc/</a>

Go to the NSSM install directory (win32 or win64) and type
> nssm.exe install

Under 'Application'
Path: <path to node installation>/node.exe
Startup directory: <path to node project application root>
Arguments: <application main>

EXAMPLE

<a href="{{ site.baseurl }}/images/2014-09-17-running-node-js-application-as-a-windows-service-using-nssm/nssm.png"><img class="alignnone size-medium wp-image-156" src="{{ site.baseurl }}/images/2014-09-17-running-node-js-application-as-a-windows-service-using-nssm/nssm.png?w=300" alt="nssm" width="300" height="150" /></a>

The remaining tabs is for further fine tuning.
