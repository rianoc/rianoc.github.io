---
layout: post
title:  "Struggles with Azure Sphere Starter Kit"
excerpt: "Struggles in setting up the Azure Sphere Starter Kit"
date:   2020-05-09 19:44:00
---

I got an [Azure Sphere MT3620 Starter Kit](https://www.element14.com/community/community/designcenter/azure-sphere-starter-kits/) in August 2019 for essentially free during a promotion.
I did little more than [claim the device](https://docs.microsoft.com/en-us/azure-sphere/install/claim-device)
 and enter a [competition](https://www.element14.com/community/docs/DOC-92683/l/sensing-the-world-challenge) which required having the device ping a server using a premade script. I never got to trying to actually use the board.

## Issue 1 - Tenants

 This April I took the board out with plans to use it for a project. I immediately ran in to issues though. The `azsphere` tool told me my user did not have any `tenants` available.

 ```cmd
azsphere login
warn: No Azure Sphere tenants found under the selected AAD user.
 ```

 I was confused as I had created one  in August. Getting guides and documentation was difficult as many links were already out of date.
 An Azure Sphere device can only be '[claimed](https://docs.microsoft.com/en-gb/azure-sphere/install/claim-device)' and linked to an account once. Without being able to figure out the tenant issue I thought I was essentially left with a paperweight.

 Luckily I found the issue. During the original setup I had forgotten that one step had been the need to create a user on [portal.azure.com](https://portal.azure.com/). This policy had since changed so when I was looking at the new documentation there was no mention of it.
 To fix I reset that users password and then could login and [migrate](https://aka.ms/AzureSphereMigration) the tenant from it to my main Microsoft account which is now allowed.

 ```cmd
 azsphere tenant migrate
 azsphere role remove-legacy-access
 ```

## Issue 2 - Enabling development

I wanted to run the [blink](https://docs.microsoft.com/en-us/azure-sphere/install/qs-blink-application) demo on the device.

One of the first steps was to enable development.

```cmd
 azsphere device enable-development
 ```

This errored for me:

 ```cmd
 error: The device is not responding. The device may be unresponsive if it is applying an Azure Sphere operating system update; please retry in a few minutes.
 ```

The [troubleshooting page](https://docs.microsoft.com/en-us/azure-sphere/install/troubleshoot-installation#device-does-not-respond) suggested deleting a JSON file or uninstall. After the JSON change did nothing I then did a reinstall which only changed the error message slightly.

```cmd
warn: The device already has the 'Enable App development' capability. No changes will be applied to its existing capabilities.
Setting device group to 'Development' with ID <devId>.
Successfully disabled application updates.
Installing debugging server to device.
Deploying 'C:\Program Files (x86)\Microsoft Azure Sphere SDK\DebugTools\gdbserver.imagepackage' to the attached device.
error: Image package failed to transfer to the device. Check if the image is too large.
error: Failed to install debugging server to device.
```

I tried some other things (adding product+group, resetting the device) but they all failed. I was again about to give up on this device when I saw a [note](https://social.msdn.microsoft.com/Forums/en-US/6403db30-e6a6-4880-8ad0-b12ebe7ba84f/error-could-not-send-device-capability-configuration-to-device?forum=azuresphere) from someone mentioning their internet had caused an issue entering development mode. This made me think about any external items which could be interfering. I had recently installed Kaspersky so I exited the application to test without it. Straight away I was able to enter development mode and deploy from Visual Studio.

## Plans

I bought a small [OLED](https://coolcomponents.co.uk/products/grove-oled-display-1-12-v2) screen and will aim to have the device report it's sensor data to Home Assistant while also displaying some other metrics.
