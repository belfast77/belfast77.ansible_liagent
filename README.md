# ansible liagent ( VMware Log Insight Agent )

#### Table of Contents

1. [Description](#description)
2. [Setup - The basics of getting started with liagent](#setup)
    * [What liagent affects](#what-liagent-affects)
    * [Setup requirements](#setup-requirements)
    * [Yum Repo Creation Optional](#yum-repo-creation-optional)
    * [Beginning with liagent](#beginning-with-liagent)
3. [Usage - Configuration options and additional functionality](#usage)
4. [Limitations - OS compatibility, etc.](#limitations)
5. [Development - Guide for contributing to the role](#development)

## Description

> vRealize Log lnsight is a highly scalable log management application with intuitive, actionable dashboards, sophisticated analytics and broad third-party extensibility. It provides deep operational visibility and faster troubleshooting across physical, virtual and cloud environments.

This role installs and configures the VMware Log Insight Agent from a custom yum repo.

## Setup

This role does not supply the VMware Log Insight Agent installation files. Installation files will need to be aquired from the Deployed VMware Log Insight Server, and the role configured to use it. Users can use yum to install these components if they're self-hosted.

If your deployed VMware Log Insight Agent has the hostname loginsight.localdomain, then your agents can be downloaded from.

https://loginsight.localdomain/admin/agents/

Once you have the agent you can setup an configure a local Yum repo, see [Yum Repo Creation Optional](#yum-repo-creation-optional)

To begin using this role, use the ansible-galaxy from the command line to install this role:
```
# ansible-galaxy install ansible-liagent
```

This will place the role into your role path.

Once the role is in place, there is just a little setup needed.

This role was designed to work with in-role defaults data.

Therefore a host_vars node deffinition file should be created in your iventory. 

For example:

/etc/ansible/inventory/your_inventory/host_vars/node1.localdomain.yml

```
liagent_srv_hostname: 'loginsight.localdomain'
liagent_service_name: 'liagentd'
liagent_package_manage: true
liagent_package: 'VMware-Log-Insight-Agent'
liagent_version: '4.7.0-9602262'
liagent_loginsight_repo: 'LogInsight_Agent'
```
### What liagent affects 

This role install the VMware Loginsight Agent Package.

It configures the agent via the config file  /var/lib/loginsight-agent/liagent.ini

It also creates a LogInsight_Agent Yum repo if required.

### Setup Requirements 

In order to Test this role against a working Log Insight Server you can setup a VMware trial for the following products:

+ VMware vSphere Hypervisor 6.7
+ https://my.vmware.com/en/web/vmware/evalcenter?p=free-esxi6
+ VMware vCenter Server Appliance 2018-10-16 | 6.7.0U1 | 3.95 GB | iso
+ https://my.vmware.com/group/vmware/evalcenter?p=vsphere-eval
+ VMware vRealize Log Insight 4.7.0 - Virtual Appliance
+https://my.vmware.com/group/vmware/evalcenter?p=vr-li


Setting this up is out of the scope of this documentation but essentially you need to setup a VMware Lab environemnt.

Install and configure VMware vSphere Hypervisor 6.7

Import and Configure VMware vCenter Server Appliance 6.7.0U1

Import VMware vRealize Log Insight 4.7.0 - Virtual Appliance via vCenter Server

+ Note a reasonable spec lab setup is required, this was tested on a
+ Intel ® NUC6I7KYK Micro Intel® 2600 MHz SOC, IRIS PRO 580. With 32GB Ram
+ Check https://www.vmware.com/resources/compatibility/search.php
+ vCenter Server Appliance - Tiny environment (up to 10 hosts or 100 virtual machines) requires 2 vCPUs & 10 GB Ram
    - The vCenter Server Appliance can be shutdown once the Loginsight appliance is installed if memory is an issue.
+ VMware vRealize Log Insight 4.7.0 - Virtual Appliance requires 4 vCPUs & 8 GB Ram
    - However in this test lab we got away with 2 vCPUs & 4 GB Ram


Further free training can be obtained with a VMware Learning Zone Basic Subscription.

https://mylearn.vmware.com/


## Yum Repo Creation Optional
Due to VMware not offering a public repo for the agent you will need to create a local repo on a simple web server (Apache, Nginx or Lighttpd) that will host the rpm in a yum repo. 

In order to have the commands createrepo and repo-sync available, install the packages createrepo and yum-utils respectively, which are not available in the default RHEL setup:

```
# yum install -y createrepo yum-utils
# mkdir -p /var/www/html/repo
# cp VMware-Log-Insight-Agent-* /var/www/html/repo
# createrepo /var/www/html/repo
```

### Beginning with liagent

Once the VMware vRealize Log Insight agent packages are hosted in the users repository the role is ready to deploy.

## Usage

If a user is installing liagent with packages provided from their local custom repo, this is the most basic way of installing liagent with default settings:

```
include ansible-liagent
```
This role uses in-role default data. The defaults directory contains the vars files.

```
defaults/
├── main.yml
└── RedHat.yml
```
liagent/defaults/main.yml

```
---
liagent_srv_hostname: default(omit)
liagent_proto: 'cfapi'
liagent_port: 9543
liagent_ssl: 'yes'
liagent_ssl_ca_path: default(omit)
liagent_reconnect: default(omit)
liagent_central_config: 'no'
liagent_debug_level: default(omit)
liagent_stats_period: default(omit)
liagent_smart_stats: 'no'
liagent_max_disk_buffer: default(omit)
liagent_logtype: default(omit)
liagent_directory: default(omit)
liagent_include: default(omit)
liagent_package_type: default(omit)
liagent_auto_update: 'no'
liagent_service_manage: true
liagent_service_ensure: true
liagent_service_enable: true
liagent_service_name: liagentd
liagent_service_provider: default(omit)
liagent_package_manage: true
liagent_package: 'VMware-Log-Insight-Agent'
liagent_version: default(omit)
```
liagent/data/RedHat.yml 

```
---
liagent_config_file: '/var/lib/loginsight-agent/liagent.ini'
liagent_logtype: 'syslog'
liagent_directory: '/var/log'
liagent_include: 'include=messages;messages.?;syslog;syslog.?'
liagent_package_type: 'rpm'
```

A node deffinition file should be created because some vales are required.
```
liagent_srv_hostname:
```
and 
```
liagent_loginsight_repo: 
```

For example:

/etc/ansible/inventory/your_inventory/host_vars/node1.localdomain.yml

```
liagent_srv_hostname: 'loginsight.localdomain'
liagent_package_manage: true
liagent_package: 'VMware-Log-Insight-Agent'
liagent_version: '4.7.0-9602262'
liagent_loginsight_repo: 'LogInsight_Agent'
```

These can be overidden in the three independent layers of configuration.

1. Extra Vars
2. Host Vars
3. Role Defaults


## Ansible Jinja2 templating lookup options

Hostname or IP address of your Log Insight server . String

`liagent_srv_hostname:` 

Manage the Loginsight Service . Boolean True or false

`liagent_service_manage:` true

Ensure the Loginsight Service files have been created. Boolean True or false

`liagent_service_ensure:` true

Ensure the Loginsight Service is enabled. Boolean True or false

`liagent_service_enable:` true

The name of the Loginsight service. String 

`liagent_service_name:` 'liagentd'

Should this role also also managed the package ? Boolean True or false

`liagent_package_manage:` true

The Name of the Loginsight Package . String

`liagent_package:` 'VMware-Log-Insight-Agent'

The Version number of the Loginsight Package . String

`liagent_version:` '4.7.0-9602262'

Name of the Loginsight repo. String

`liagent_loginsight_repo:` 'LogInsight_Agent'

Path to the Loginsight config file . Absolutepath . Default: omit

`liagent_config_file_` default(omit)

Protocol can be cfapi (Log Insight REST API), syslog. Default:

`liagent_proto:` 'cfapi'

Log Insight server port to connect to. Default ports for protocols (all TCP):
syslog: 514; syslog with ssl: 6514; cfapi: 9000; cfapi with ssl: 9543. Default:

`liagent_port:` 9543

Is SSL to be used ? String

`liagent_ssl:` 'yes'

Path to the ssl_ca file . Absolutepath . Default: omit

`liagent_ssl_ca_path:` default(omit)

Time in minutes to force reconnection to the server.

This option mitigates imbalances caused by long-lived TCP connections. Default: omit

`liagent_reconnect:` default(omit) 

Allow the agent to receive central configuration from the server.
If disabled, only agent-side configuration will be applied. Default: No

`liagent_central_config:` 'no'

Time in minutes to force reconnection to the server.

This option mitigates imbalances caused by long-lived TCP connections. Default: omit

`liagent_debug_level:` default(omit)

Frequency to print agent dynamic information in minutes. Default: omit

`liagent_stats_period:` default(omit) 

Allow the agent to automatically decrease dynamic information print period to 1 minute
in case if agent abnormal performance is detected.
If disabled, agent performance won't be monitored at all. StringDefault:  No

`liagent_smart_stats:` 'no'

Max local storage usage limit (data + logs) in MBs. Valid range: 100-2000 MB.

`liagent_max_disk_buffer:` default(omit)

Uncomment the appropriate section to collect system logs
The recommended way is to enable the Linux content pack from LI server. String  Default: syslog

`liagent_logtype:` default(omit)

Path to the log file directory . Absolutepath . Default: /var/log

`liagent_directory:` default(omit)

List of files to be read in the "liagent_directory" . String

`liagent_include:` default(omit)

Type of Package . String . Default : omit

`liagent_package_type:` default(omit)

Enable automatic update of the agent. If enabled:
the agent will silently check for updates from the server and
if available will automatically download and apply the update.

`liagent_auto_update:` 'no'

 Default service manager on the system.
 
`liagent_service_manager:` default(omit)


## Limitations

This Version is compatible with:

Operating Systems:

RedHat: 6, 7

CentOS: 6, 7

## Development

Currently tested manually on Centos 7, but will eventually add automated testing and are targeting compatibility with other platforms.

Tested with ansible 2.7.8

## VMware Tutorial Videos

Here are some great tutorials from [sysadmintutorials.com](https://www.sysadmintutorials.com) to setup all that you need to get started with setting up a test lab for this role.

[![vSphere 6.7 - How to install and configure VMware ESXi 6.7](http://img.youtube.com/vi/AQLFQW0GvV0/0.jpg)](https://www.youtube.com/watch?v=AQLFQW0GvV0)

[![vSphere 6.7 - How to install and configure VMware vCenter 6.7 Appliance](http://img.youtube.com/vi/U-rilkWMkO4/0.jpg)](http://www.youtube.com/watch?v=U-rilkWMkO4)

[![vSphere 6.5 - How to install and configure VMware vRealize Log Insight 4](http://img.youtube.com/vi/aJeTBO_rWms/0.jpg)](http://www.youtube.com/watch?v=aJeTBO_rWms)



## Example Playbook

This is an example of how you might include this role in a playbook.

```
---
- name: ansible_liagent play
  hosts: webservers
  become: yes
  roles: 
    - belfast77.ansible_liagent
```

## License

BSD

## Contributors

Contributers very, very welcome and I'll buy you a beer if we meet at Ansible Automates or meetups.
# ansible-liagent
