+++ 
draft = false
date = 2025-03-07T19:54:06+11:00
title = "Network diagnostics in Azure App Services"
description = "Some reminders about the tools and commands that can/should be used within kudu when troubleshooting App Services"
slug = ""
authors = ["Matthew Macdonald"]
tags = [
    "azure", "networking", "troubleshooting"
]
+++

# Introduction

Over the years I've found myself having to debug network problems within app services. Now the most fool proof way to do this is to spin up a vm inside the subnet or vnet in question, giving you access to the full range of tooling you'd want to use. This isn't always possible though - the app service in question may be running in production, where you can't spin up new resources at a whim. Or maybe you want to simulate connectivity coming from your app service as truly as possible.

Ala Kudu. This management environment gives you access to a multitude of tools and information about your app service - which most cloud engineers would be aware of.

The functionality we are most interested in this case though, is the interactive shell. The problem with this shell however, is that the tools you are used to using, such as `ping` & `tracert` do not work like they should.

Below, I will outline the methods of getting decent diagnostics from within the Kudu environment for each OS type.

> Note that this is not applicable to container based web apps - in which case you can just ssh or remote exec into the container.

# Windows

Once you are in your web based shell (CMD or PowerShell), trying to use commands like `ping` or `tracert` will just throw errors.

As per the MS documentation referenced in the Links section at the bottom of this post, they have disabled these commands for security reasons, They have created some tools in their place though - which achieve the same thing.

### Ping

In place of `ping` use `tcpping.exe`.

The syntax is as follows: `tcpping.exe hostname [optional: port]`

### DNS Lookup

`nslookup` actually works here, so just use that if you wish. The docs do suggest however to use the following:

`nameresolver.exe hostname [optional:DNS Server]`

### Http calls

At time of writing, I tried `curl` & `Invoke-WebRequest` which both failed.

`Invoke-RestMethod <url>` works though - so use that.

# Linux

### Ping

In place of `ping` use `tcpping`.

The syntax is as follows: `tcpping hostname [optional: port]`

### DNS

Nameresolver is not available in linux. So use nslookup.

`nslookup hostname [optional:DNS Server]`

### Http calls

`curl` is fully functional in kudu linux. So use to your hearts content.

# Further Reading

- [Windows network watcher](https://learn.microsoft.com/en-us/azure/network-watcher/network-watcher-overview?wt.mc_id=AZ-MVP-5005255) looks promising for network diagnostics - but I have not used it.

# Links

- [Troubleshoot virtual network integration with Azure App Service](https://learn.microsoft.com/en-us/troubleshoot/azure/app-service/troubleshoot-vnet-integration-apps)
