# ISMtools for using ISM REST API

## Important: This toolset is now available at [Github of Fujitsu](https://github.com/fujitsu/ISMtools).
### This site will no longer be maintained!
---

Here are some basic scripts to use the REST API for Infrastructure Manager (ISM)  
The number of tools/scripts will increase over time ...

This toolset is provided W/O ANY WARRANTY and use at your own risk!  

## Requirements

This toolset is intended to be used in Linux environments. Alternatively it can be used in Windows environments with activated WSL (Windows Subsystem Linux) and installed Ubuntu or Debian for example from Microsoft Store. You can also use [Cygwin](https://cygwin.org).

Following commands are required:
- bash
- curl
- sed
- optional jq (to get pretty output / needed for some scripts like ism_del_iso)

## Setup

To use this toolset run the following steps:

### 1. Clone the repository

```shell
$ git clone https://github.com/JuergenOrth/ISMtools.git
$ cd ISMtools
```
or extract this [ZIP file](https://github.com/JuergenOrth/ISMtools/archive/refs/heads/master.zip).

### 2. Edit the configfile

Set the variables `ISM`, `USER` and `PASSWD` by editing *.ism_env* at least. 
Alterntively you can define environment vars `ISM_VA` and `ISM_CRED` to define the ISM virtual appliance address and user credentials respectivlely.

Syntax:
```
ISM_VA=[ISM_IP|ISM_HOSTNAME|ISM_FQDN][:ISM_PORT]
e.g.
ISM_VA=myism.example.com	# Port will be $ISM_PORT in this case
ISM_VA=192.168.1.2:1433
ISM_VA=:433

ISM_CRED=[USER][:PASSWORD]
e.g.
ISM_CRED=:verysecret	# user will be $USER in this case
ISM_CRED=smith:somesecret
ISM_CRED=foo		# password will be $PASSWD in this case
```

```shell
$ chmod go-rwx .ism_env	# for security
$ vi .ism_env		# or any other editor you prefer
```

## Commands
### 1. ism_cmd  
Basic script to run REST call with different methods like GET, POST, PATCH, ..  
Usage: `ism_cmd <method> <rest_endpoint> [<additional_params ..>]`  

### 2. ism_gfupdate  
Update complete firmware/driver repository of ISM from GlobalFlash. Please consider to have GlobalFlash enabled in ISM. It take some minutes for downloading the current list of files. After that a download is iniated as task within ISM.  
Usage: `ism_gfupdate`


## Examples
1. Check if GlobalFlash is enabled (check end of output line: Mode should be true)

```shell
$ ./ism_cmd get /system/settings/firmware/ftsfirmware
{"SchemaType":"https://192.168.3.203:25566/ism/schema/v2/System/SystemSettingsFirmwareFtsFirmware-GET-Out.0.0.1.json","MessageInfo":[],"IsmBody":{"Mode":true}}
```
2. Perform complete update of ISM repository from GlobalFlash
```shell
$ ./ism_gfupdate
Retrieving downloadable firmware/driver - Please wait ~2 minutes ... done
Saving downloadable firmware/driver list.
Starting download ...
{
  "SchemaType": "https://192.168.3.203:25566/ism/schema/v2/System/SystemSettingsFirmwareFtsFirmwareDownload-POST-Out.0.0.1.json",
  "MessageInfo": [],
  "IsmBody": {
    "TaskId": "21",
    "CancelUri": "https://192.168.3.203:25566/ism/api/v2/system/settings/firmware/ftsfirmware/download/cancel"
  }
}

```
To perform automatic updates on a regular basis you can add a line like  
`0 21 * * * ~/ISMtools/ism_gfupdate`  
to your crontab file in order to run the script each evening at 9pm.  
