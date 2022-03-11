# ISMtools for using ISM REST API

Here are some basic scripts to use the REST API for Infrastructure Manager (ISM)

## Requirements

This toolset is intendet to be used in Linux environments. Alternatively it can be used in Windows environment where you can activate WSL (Windows Subsystem Linux) and e.g. install Ubuntu or Debian from Microsoft Store. You can also use [Cygwin](https://cygwin.org).

Following commands are required:
- bash
- sed
- curl
- optional jq (to get pretty output)

## Setup

To use this toolset run the following steps:

### 1. Clone the repository

```shell
git clone https://github.com/JuergenOrth/ISMtools.git
cd ISMtools
```
or extract this [ZIP file](https://github.com/JuergenOrth/ISMtools/archive/refs/heads/master.zip).

### 2. Edit the configfile

Set the variables `ISM_IP`, `USER` and `PASSWD` by editing *.ism_env* at least. 


```shell
vi .ism_env		# or any other editor you prefer
chmod go-rwx .ismenv 	# for security
```

### 3. Certificate assignment for Infrastructure Manager

Acquire the certificate used for https communication and assign in the Ansible server.
For information on how to assign certificates, refer to "3.1.1 Preparation" in
"FUJITSU Software Infrastructure Manager V2.2 REST API Reference Manual".

### 4. Setting config file

Set the account information of Infrastructure Manager for the confg file (`ism_config.json`).
Execute `config_setting.sh` with Ansible server as follows.The config file is created in the current directory when executing it.  
Refer from [[Note1]](#note-1) to [[Note5]](#note-5).

#### Example

```shell
$ cd /etc/ansible/ism-ansible/
$ chmod +x ./config_setting.sh
$ ./config_setting.sh

> Enter IP address or FQDN:
  <IP address> or <FQDN>
> Enter port number:
  <Port No>
> Enter user name:
  <Login user name>
> Enter password:
  <Login password>
> Enter full path of certificate file:
  <full path of certificate file>
> completed
```

In each example in the above < >, specify the information of Infrastructure Manager.  
Refer to [[Note6]](#note-6).

Confirm the config file was created.

#### Example of execution with IP address

```shell
$ cat ./ism_config.json
  {"ip":"192.168.1.10","portNo":"25566",
    "credentials":{"userName":"administrator",
    "password":"U2FsdGVkX18iGtLsngKkgWtQQ3+j0s5W1aSTizoWny8="},
    "certificate": "/etc/ansible/ism-ansible/certificate.crt"}
```

Describe the path of the config file in playbook.  
Refer to [[Note7]](#note-7) and [[Note8]](#note-8).

#### Playbook example

```yaml
- name: Firmware update
  hosts: servers
  connection: local

  tasks:
   - name: Execution of ism_firmware_update
     ism_firmware_update:
       config: "/etc/ansible/ism-ansible/ism_config.json"
```

### 5. Setting inventory file

The inventory file describes the IP address (OS, hardware) and hostname (OS, hardware) of the operation target node.
Specify the operation target for the inventory file.  
Allocate the inventory file in Ansible server.

#### INI Format

```INI
[<Host group name1>]
<Operation target node1> ism_profile_name=<profile name>
<Operation target node2> ism_node_name=<Node Name> ism_node_type=<Node type> ism_node_model=<Model name>
<Operation target node3> ism_profile_name=<Profile name> ism_computer_name=<Computer name> ism_os_ip_address=<OS IP Address>
<Operation target node4>
...
[<Host group name2>]
<Operation target server>
```

#### YAML Format

```YAML
all:
  children:
    <Host group name1>:
      hosts:
        <Operation target node1>:
          ism_profile_name: <Profile name>
        <Operation target node2>:
          ism_node_name: <Node name>
          ism_node_type: <Node type>
          ism_node_model: <Model name>
        <Operation target node3>:
          ism_profile_name: <Profile name>
          ism_computer_name: <Computer name>
          ism_os_ip_address: <OS IP Address>
        <Operation target node4>:
        ...
    <Host group name2>:
      hosts:
        <Operation target server>:
```

#### Settings

<table>
<tbody>
<tr>
  <th>Name</th>
  <th>Value</th>
  <th>Remarks</th>
</tr>
<tr>
  <td>Host group name</td>
  <td>Any host group name</td>
  <td>Specifies an arbitrary host group name.<br>
      The host group is defined the multiple hosts as one group.<br>
      In Ansible, the playbook can be executed for the host group unit.
  </td>
</tr>
<tr>
  <td>Operation target node</td>
  <td>One of the following<br>
  - IP address<br>
  - Host name (FQDN)</td>
  <td>Multiple specifications are available.<br>
  Specifies the IP address of the operation target node registered in Infrastructure Manager or the host name (FQDN) for its IP address.<br>When the OS information of the operation target node is registered in Infrastructure Manager, the host name (FQDN) for the IP address of the OS information or the IP address can be specified.<br>
  <a href="#note-9">[Note9]</a> <a href="#note-10">[Note10]</a> <a href="#note-11">[Note11]</a></td>
</tr>
<tr>
  <td>Profile name</td>
  <td>The name of the profile to apply to the operation target node.<br>
  Or, the new profile name when you copy the profile.</td>
  <td>
    Optional.<br>
    Specifies to run the profile assignment, copy profile module.<br>
    Otherwise, it is ignored.<br>
    If omitted, no key part ("ism_profile_name=", "ism_profile_name:") should be specified.
   </td>
</tr>
<tr>
  <td>OS IP Address</td>
  <td>The OS IP address to specify when copying the profile</td>
  <td>Optional.<br>Specifies to run the copy profile module.<br>Otherwise, it is ignored.<br>If omitted, no key part ("ism_os_ip_address=","ism_os_ip_address:") should be specified.</td>
</tr>
<tr>
  <td>Computer name</td>
  <td>Computer name to specify when copying the profile</td>
  <td>Optional.<br>Specifies to run the copy profile module.<br>Otherwise, it is ignored.<br>If omitted, no key part ("ism_computer_name=","ism_computer_name:") should be specified.</td>
</tr>
<tr>
  <td>Operation target server</td>
  <td>One of the following<br>
  - IP address<br>
  - Host name (FQDN)</td>
  <td>Specifies the IP address of the ISM server or the hostname (FQDN) for its IP address.<br>
      Use this value when performing unique operations with ISM, such as updating downloadable firmware information.
</tr>
<tr>
  <td>Node name</td>
  <td>The name of the node to be registered with the operation target node</td>
  <td>Optional.<br>Specifies to run the node registration module.<br>It is ignored for modules other than node registration.<br>If omitted, no key part ("ism_node_name=","ism_node_name:") should be specified.
  </td>
</tr>
<tr>
  <td>Node type</td>
  <td>Node type to be registered in the operation target node</td>
  <td>Optional<br>Specifies to run the node registration module.<br>It is ignored for modules other than node registration.<br>If omitted, no key part ("ism_node_type=","ism_node_type:") should be specified.</td>
</tr>
<tr>
  <td>Model name</td>
  <td>Model name to be registered in the operation target node</td>
  <td>Optional.<br>Specifies to run the node registration module.<br>It is ignored for modules other than node registration.<br>If omitted, no key part ("ism_node_model=","ism_node_model:") should be specified.
  </td>
</tr>
</tbody>
</table>

#### Character code

UTF-8

#### File Location

Anywhere on the management server(Ansible).

#### Example

- Host group name: servers
  - Operation node1
    - IP address of OS information: 192.168.1.11
    - Profile name: profileA
  - Operation node2
    - Node IP address: 192.168.1.12
  - Operation node3
    - Host name for IP address of OS information: node3-os.test.local
  - Operation node4
    - Host name for IP address of node: node4
  - Operation node5
    - Node IP address: 192.168.1.13
    - Profile name: profileB
    - Computer name: node5
    - Host name for IP address of OS information: 192.168.1.23
- Host group name: ism_server
  - Operation server: 192.168.1.10

##### INI Format

```INI
[servers]
192.168.1.11 ism_profile_name=profileA
192.168.1.12 ism_node_name=nodeA ism_node_type=server ism_node_model="PRIMERGY RX300S8"
node3-os.test.local
node4
192.168.1.13 ism_profile_name=profileB ism_computer_name=node5 ism_os_ip_address=192.168.1.23

[ism_server]
192.168.1.10
```

##### YAML Format

```YAML
all:
  children:
    servers:
      hosts:
        192.168.1.11:
          ism_profile_name: profileA
        192.168.1.12:
          ism_node_name: node_A
          ism_node_type: server
          ism_node_model: PRIMERGY RX300 S8
        node3-os.test.local:
        node4:
        192.168.1.13:
          ism_profile_name: profileB
          ism_computer_name: node5
          ism_os_ip_address: 192.168.1.23
    ism_server:
      hosts:
        192.168.1.10:
```

### 6. Modules and license check  

The following is a list of modules available and unavailable in Essential.  

<table>
<tbody>
<tr>
  <th rowspan="2">Module name</th>
  <th colspan="2">Operation Mode<a href="#note-13"> [Note13]</a></th>
</tr>
<tr>
  <th>Other than Essential<a href="#note-14"> [Note14]</a></th>
  <th>Essential</th>
</tr>

<tr>
  <td>ism_backup.py</td>
  <td align="center">Yes</td>
  <td align="center">Yes</td>
</tr>
<tr>
  <td>ism_copy_profile.py</td>
  <td align="center">Yes</td>
  <td align="center">No</td>
</tr>
<tr>
  <td>ism_download_firmware.py</td>
  <td align="center">Yes</td>
  <td align="center">Yes</td>
</tr>
<tr>
  <td>ism_firmware_list.py</td>
  <td align="center">Yes</td>
  <td align="center">Yes</td>
</tr>
<tr>
  <td>ism_firmware_update.py</td>
  <td align="center">Yes</td>
  <td align="center">Yes</td>
</tr>
<tr>
  <td>ism_get_download_firmware_list.py</td>
  <td align="center">Yes</td>
  <td align="center">Yes</td>
</tr>
<tr>
  <td>ism_get_inventory_info.py</td>
  <td align="center">Yes</td>
  <td align="center">Yes</td>
</tr>
<tr>
  <td>ism_get_power_status.py</td>
  <td align="center">Yes</td>
  <td align="center">Yes</td>
</tr>
<tr>
  <td>ism_get_profile_info.py</td>
  <td align="center">Yes</td>
  <td align="center">No</td>
</tr>
<tr>
  <td>ism_get_report_info.py</td>
  <td align="center">Yes</td>
  <td align="center">Yes</td>
</tr>
<tr>
  <td>ism_maintenance_mode_setting.py</td>
  <td align="center">Yes</td>
  <td align="center">Yes</td>
</tr>
<tr>
  <td>ism_power_on.py</td>
  <td align="center">Yes</td>
  <td align="center">Yes</td>
</tr>
<tr>
  <td>ism_profile_assignment.py</td>
  <td align="center">Yes</td>
  <td align="center">No</td>
</tr>
<tr>
  <td>ism_refresh_node_info.py</td>
  <td align="center">Yes</td>
  <td align="center">Yes</td>
</tr>
<tr>
  <td>ism_retrieve_download_firmware_info.py</td>
  <td align="center">Yes</td>
  <td align="center">Yes</td>
</tr>
</tbody>
</table>

Yes:Available  
No:Unavailable  
[[Note15]](#note-15)  

### 7. Playbook execution

The following commands are executed with Ansible server.

```shell
ansible-playbook <playbook file path> -i <inventory file path>
```

#### Example

playbook file path: `/etc/ansible/ism-ansible/examples/ism_firmware_list.yml`  
inventory file path: `/etc/ansible/hosts`  

```shell
ansible-playbook /etc/ansible/ism-ansible/examples/ism_firmware_list.yml -i /etc/ansible/hosts
```

## Report information comparison tool

A tool that compares the output files of the report information retrieval module.

### Deployment path

By default, it is placed in the following path:  
\<Directory checked out from Git>/ism-ansible/tools/ism_report_diff.py

### Tool execution command

```shell
python  ism_report_diff.py <FILE1> [<FILE2>]
```

### Options

<table>
<tbody>
<tr>
  <th>Parameter</th>
  <th align="center">Required</th>
  <th>Description</th>
</tr>
<tr>
  <td>FILE1</td>
  <td align="center">Yes</td>
  <td>Specifies an old report information file to be compared.</td>
</tr>
<tr>
  <td>FILE2</td>
  <td align="center">No</td>
  <td>Specifies a new report information file to compare with.<br>
If this parameter is omitted, FILE1 and FILE2 are not compared and only the information of the file specified in FILE1 is output.
</td>
</tr>
</tbody>
</table>

### Examples

- To compare the old and new report information files

```shell
python ism_report_diff.py 2019-11-04_14-35-31.json 2019-12-04_09-01-02.json
```

- To output the information of only FILE1 without comparison

```shell
python ism_report_diff.py 2019-12-04_09-01-02.json
```

### Output

<table>
<tbody>
<tr>
  <th colspan="4">Name</th>
  <th>Description</th>
</tr>
<tr>
  <td colspan="4">Total</td>
  <td>Number of nodes registered in ISM.</td>
</tr>
<tr>
  <td colspan="4">Diff</td>
  <td>Number of nodes added and deleted by comparing the old and new report information files<br>
  <a href="#note-16">[Note16]</a>

Format:<br>
+&lt;Number of adding nodes>, -&lt;Number of deleting nodes><br>

Example: Adding two nodes and deleting one node<br>
+2, -1</td>
</tr>
<tr>
  <td colspan="4">Status</td>
  <td>Number of nodes in each status.</td>
</tr>
<tr>
  <td rowspan="5"></td>
  <td colspan="3">Error</td>
  <td>Number of nodes that the status is Error.</td>
</tr>
<tr>
  <td colspan="3">Warning</td>
  <td>Number of nodes that the status is Warning.</td>
</tr>
<tr>
  <td colspan="3">Unknown</td>
  <td>Number of nodes that the status is Unknown.</td>
</tr>
<tr>
  <td colspan="3">Updating</td>
  <td>Number of nodes that the status is Updating.</td>
</tr>
<tr>
  <td colspan="3">Normal</td>
  <td>Number of nodes that the status is Normal.</td>
</tr>
<tr>
  <td colspan="4">Alarm status</td>
  <td>Number of nodes in each alarm status</td>
</tr>
<tr>
  <td rowspan="4"></td>
  <td colspan="3">Error</td>
  <td>Number of nodes that the alarm status is Error.</td>
</tr>
<tr>
  <td colspan="3">Warning</td>
  <td>Number of nodes that the alarm status is Warning.</td>
</tr>
<tr>
  <td colspan="3">Info</td>
  <td>Number of nodes that the alarm status is Info.</td>
</tr>
<tr>
  <td colspan="3">Normal</td>
  <td>Number of nodes that the alarm status is Normal.</td>
</tr>
<tr>
  <td colspan="4">Added node</td>
  <td>Added node information for old and new report information files<br><a href="#note-16"></td>
</tr>
<tr>
  <td rowspan="5"></td>
  <td colspan="3">Node &lt;N></td>
  <td>Node Information <br>&lt;N> is an integer, starting with 1</td>
</tr>
<tr>
  <td colspan="1" rowspan="4"></td>
  <td colspan="2">Name</td>
  <td>Node name</td>
</tr>
<tr>
  <td colspan="2">Type</td>
  <td>Node type</td>
</tr>
<tr>
  <td colspan="2">Model</td>
  <td>Model name</td>
</tr>
<tr>
  <td colspan="2">IP</td>
  <td>IP address<br>Displays None for nodes that do not have an IP address</td>
</tr>
<tr>
  <td colspan="4">Firmware</td>
  <td>Firmware Information that can be updated.<br>
      If there is no firmware that can be updated, it will not be output.
  </td>
</tr>
<tr>
  <td rowspan="15"></td>
  <td colspan="3">Node &lt;N></td>
  <td>Node Information where existing the updatable firmware.<br>
&lt;N> is an integer, starting with 1</td>
</tr>
<tr>
  <td rowspan="14"></td>
  <td colspan="2">Name</td>
  <td>Node name</td>
</tr>
<tr>
  <td colspan="2">Type</td>
  <td>Node type</td>
</tr>
<tr>
  <td colspan="2">Model</td>
  <td>Model name</td>
</tr>
<tr>
  <td colspan="2">IP</td>
  <td>IP address<br>
  Displays None for nodes that do not have an IP address.</td>
</tr>
<tr>
  <td colspan="2">Firmware &lt;N></td>
  <td>Currently applied firmware information.<br>
&lt;N> is an integer, starting with 1</td>
</tr>
<tr>
  <td rowspan="3"></td>
  <td>Name</td>
  <td>Firmware name</td>
</tr>
<tr>
  <td>Type</td>
  <td>Firmware type</td>
</tr>
<tr>
  <td>Version</td>
  <td>Version</td>
</tr>
<tr>
  <td colspan="2">Updatable firmware &lt;N></td>
  <td>Firmware information that can be updated<br>
&lt;N> is an integer, starting with 1</td>
</tr>
<tr>
  <td rowspan="5"></td>
  <td>Name</td>
  <td>Firmware Name</td>
</tr>
<tr>
  <td>Type</td>
  <td>Firmware type</td>
</tr>
<tr>
  <td>Version</td>
  <td>Version</td>
</tr>
<tr>
  <td>RepositoryName</td>
  <td>Repository</td>
</tr>
<tr>
  <td>OperationMode</td>
  <td>Supported mode</td>
</tr>
</tbody>
</table>

### Notes

- Sample playbook using ism_report_diff.py  
  [examples/ism_diff_report.yml](/examples/ism_diff_report.yml)

## Changing processing time of Update Firmware and Profile Assignment settings

In the module of Update Firmware or Profile Assignment, waiting to complete the task in the internal processing.  
The timeout period for the process of waiting to complete the task can be changed with the following ism_user_settings file.  
Refer to [[Note12]](#note-12).

ism_user_settings file path: `/etc/ansible/ism-ansible/module_utils/ism_user_settings.py`  

### Example

Timeout period of firmware update: `14400` (second) (= 4hours)  
Timeout period of application of profile: `18000` (second) (= 5hours)

```python
#!/usr/bin/python
#coding: UTF-8

# user_settings_value

'''
Set time confirmation for task confirmation
'''
FIRMWARE_UPDATE_TIME_OUT = 14400

PROFILE_ASSIGNMENT_TIME_OUT = 18000
```

## Notes

<a name="note-1">[Note1]  
When the config file already exists, it will be overwritten.

<a name="note-2">[Note2]  
Specify the input values in single-byte upper-case, lower-case alphabetic letters, numbers, and symbols.

<a name="note-3">[Note3]  
When re-setting by changing the input information, make sure that the playbook and module are stopped, and that the proper procedure is executed again.

<a name="note-4">[Note4]  
When FQDN is used for the connection with Infrastructure Manager, confirm beforehand whether FQDN of Infrastructure Manager is available for name resolution on the Ansible server.

<a name="note-5">[Note5]  
Presently IPv6 is not supported. For the connection with Infrastructure Manager, specify the IP address of IPv4 or host name (FQDN) that are available for name resolution of IPv4.

<a name="note-6">[Note6]  
If the following error occurs when FQDN of Infrastructure Manager is specified with `config_setting.sh`, name resolution of FQDN of Infrastructure Manager has failed on the Ansible server. Ensure the action for name resolution and re-execute.

Example (For FQDN ism.test.local):

```shell
$ ./ism-ansible/config_setting.sh
Enter IP address or FQDN:
ism.test.local
Traceback (most recent call last):
  File "<string>", line 1, in <module>
socket.gaierror: [Errno -2] Name or service not known
```

<a name="note-7">[Note7]  
Specify the config path in single-byte upper-case, lower-case alphabetic letters, numbers and symbols.

<a name="note-8">[Note8]  
Ansible Modules for Infrastructure Manager is executed on the Ansible server. Therefore, the specification for "Connection: local" of the playbook is mandatory.

<a name="note-9">[Note9]  
Presently IPv6 is not supported. Specify the IP address of IPv4 or host name (FQDN) that are available for name resolution of IPv4.

<a name="note-10">[Note10]  
When the host name (FQDN) is specified, confirm beforehand whether the host name (FQDN) is available for name resolution on the Ansible server is confirmed.

<a name="note-11">[Note11]  
When operating the node registration module, specify the target server to operate by specifying the IP or host name (FQDN) of iRMC.

<a name="note-12">[Note12]  
Since the time-out time of 10800(second)(For = three hours), which sufficient for waiting for the task to complete, is specified in the default setting, it is not normally required to be changed.

<a name="note-13">[Note13]  
This is the "Operation Mode" value output by "ismadm license show" command.

<a name="note-14">[Note14]  
The "Operation Mode" other than Essential are as follows.  

- Advanced  
- Advanced (Trial)  
- Advanced for PRIMEFLEX  
- Advanced for PRIMEFLEX (Trial)  
- NFLEX  
- NFLEX (Trial)  

<a name="note-15">[Note15]  
The following error is output when executing an unavailable module.  
`This module is not supported on Essential mode.`

<a name="note-16">[Note16]  
Output only if FILE2 is specified as a tool option.

## Software support

The software in this project is not supported.

## License

This project is licensed under the GPL v3 license. See the [LICENSE](LICENSE) for more information.

## Copyright

Copyright FUJITSU LIMITED 2017-2020

## API

- FUJITSU Software Infrastructure Manager REST API Reference Manual  
<http://www.fujitsu.com/jp/products/software/infrastructure-software/infrastructure-software/serverviewism/technical/>
