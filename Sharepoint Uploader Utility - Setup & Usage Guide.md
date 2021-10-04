# Sharepoint Uploader Utility - Setup & Usage Guide

Author: Nelson Roque (nelson.roque@ucf.edu)

## Overview

Depending on the data needs of a project, there may be a desire to mirror data in multiple locations. Scott Yabiku has created a command line utility to upload data to Microsoft Sharepoint sites. This guide will walk you through configuration of the utility, including necessary setup within your Sharepoint site.

## Getting Started

In sections below, you will notice [VARIABLES] in square brackets - replace with the information relevant to your installation and process.

### Setup Utility on Linux Machine

#### Motivation

The motive is to use the command line to transfer large files from a Linux system to a SharePoint folder, such as when archiving files during a cron job. Individual user authentication is not supported. Instead, you must have previously created an App registration and recorded the AppId (also called ClientId) and Secret. See https://docs.microsoft.com/en-us/sharepoint/dev/solution-guidance/security-apponly-azureacs.

#### Building

- Install .NET 5.0 SDK from https://dotnet.microsoft.com/download. Detailed info at https://docs.microsoft.com/en-us/dotnet/core/install/
- Clone this repo
- Build the app:
  - `dotnet build --configuration Release`
- Publish to a single executable. If on Linux:
  - `dotnet publish --configuration Release -r linux-x64 -p:PublishSingleFile=true --self-contained true`
- Executable is in folder bin/Release/net5.0/linux-x64/publish

For more information, visit the Scott Yabiku's Github repository for the [sharepoint-upload](https://github.com/syabiku/sharepoint-upload) tool.

#### Params

```
-u, --url            Required. Sharepoint site url.

-a, --appid          Required. App Id.

-s, --secret         Required. App secret.

-f, --folder         Required. SharePoint destination folder path.

--help               Display this help screen.

--version            Display version information.

filename             Required. Local filename to upload, including path

```

### Configure Application on Sharepoint

1. Create an app inside of the Sharepoint site
   1. https://[TENANT].sharepoint.com/sites/[SITE_NAME]/_layouts/15/AppRegNew.aspx 
2. Approve the app
   1. https://[TENANT].sharepoint.com/sites/[SITE_NAME]/_layouts/15/appinv.aspx
   2. Add XML below for appropriate write permissions. May require Tenant administrator approval.

   ```
   <AppPermissionRequests AllowAppOnlyPolicy="true"> 
   <AppPermissionRequest Scope="http://sharepoint/content/sitecollection"
    Right="Write" />
</AppPermissionRequests>

   ```


### Configure CRON Job

```
#!/bin/bash

# get date time
Datetime=$(date +"%Y_%m_%d__%H_%M_%S")

# cd to data directory
echo "changing directory"
cd /var/www/html/[DIRECTORY]

# zip
echo "zipping"
sudo zip -r "/tmp/[ZIP_PREFIX]_$Datetime.zip" ./eas_followups

# send to sharepoint
echo "sending to sharepoint"
sudo /usr/local/sbin/sharepoint-upload -u 'https://[TENANT].sharepoint.com/sites/[SITE_NAME]/' -a '[APP_ID]' -s '[APP_SECRET]' -f 'Scripted Uploads' "/tmp/[ZIP_PREFIX]_$Datetime.zip"
if [ $? -ne 0 ]
then
    mail -s "Backup of data failed" '[EMAILS TO NOTIFY ON ERROR - comma seperated]' <<< " "
fi

# delete zip
echo "deleting zip"
sudo rm -rf "/tmp/[ZIP_PREFIX]_$Datetime.zip"
```