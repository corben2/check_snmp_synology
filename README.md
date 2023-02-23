# synology-nrpe-plugin
A simple NRPE Plugin which checks the health of a Synology NAS

It supports checking the following items:
* System status
* Power status
* Fan status
* Disks status
* RAID (Volumes) status
* DSM version and update status
* System temperature
* UPS information (maybe)

## Based on
This plugin is a modified version of a plugin by deegan199. It can be found [here.](https://exchange.nagios.org/directory/Plugins/Network-and-Systems-Management/Others/Synology-status/details). It was modified by Tobias Schenke, Nicolas Ordonez, Corben Eastman and Fabian Ihle. 

### Changes From Original
This version supports volume usage on DSM 6.2 and 7.0. Also, it checks readings by type,
rather then checking everything at once. This allows for splitting checks into
different services in Nagios / Icinga. Look below for more information. As a part of this
change, the verbose option was removed and it is now always verbose. Also, the
UPS option was removed and added as a check type instead (but it hasn't been tested with a UPS).


## Requirements
snmpwalk and snmpget need to be installed. Also, the SNMP agent on the NAS has to be activated.

## Usage
```sh
usage: ./check_snmp_synology [OPTIONS] -u [user] -p [pass] -h [hostname]
options:
            -u [snmp username]          Username for SNMPv3
            -p [snmp password]          Password for SNMPv3
			-l [level]            Set security level (noAuthNoPriv|authNoPriv|authPriv) (default AuthPriv)
			-a [protocol]         Set authentication protocol (MD5|SHA) (default SHA)
			-x [protocol]         Set privacy protocol (DES|AES) (default AES)

            -2 [community name]         Use SNMPv2 (no need user/password) & define community name (ex: public)

            -h [hostname or IP](:port)  Hostname or IP. You can also define a different port

            -W [warning temp]           Warning temperature (for disks & synology) (default 50)
            -C [critical temp]          Critical temperature (for disks & synology) (default 60)

            -w [warning %]              Warning storage usage percentage (default 80)
            -c [critical %]             Critical storage usage percentage (default 95)

            -i                          Ignore DSM updates
            -U                          Show informations about the connected UPS (only information, no control)
            -v                          Verbose - print all informations about your Synology

examples:       ./check_snmp_synology -u admin -p 1234 -h nas.intranet
                ./check_snmp_synology -u admin -p 1234 -h nas.intranet -v
                ./check_snmp_synology -2 public -h nas.intranet
                ./check_snmp_synology -2 public -h nas.intranet:10161
```

## Nagios Configuration Examples
### Command Definition
```
define command{
        command_name    synology_check
        command_line    /usr/lib64/nagios/plugins/check_snmp_synology -2 public -h $HOSTADDRESS$ -t $ARG1$
}
```

### Service Definitions
```
define service {
	use	                   generic-service
	hostgroup_name		   synology
	service_description	   System
	check_command		   synology_check!system
	notifications_enabled      1
        check_interval             5
}

define service {
	use	                   generic-service
	hostgroup_name		   synology
	service_description	   Version
	check_command		   synology_check!version
	notifications_enabled      1
        check_interval             5
}

define service {
	use	                   generic-service
	hostgroup_name		   synology
	service_description	   Power
	check_command		   synology_check!power
	notifications_enabled      1
        check_interval             5
}

define service {
	use	                   generic-service
	hostgroup_name		   synology
	service_description	   Fans
	check_command		   synology_check!fan
	notifications_enabled      1
        check_interval             5
}

define service {
	use	                   generic-service
	hostgroup_name		   synology
	service_description	   Disks
	check_command		   synology_check!disk
	notifications_enabled      1
        check_interval             5
}

define service {
	use	                   generic-service
	hostgroup_name		   synology
	service_description	   RAID
	check_command		   synology_check!raid
	notifications_enabled      1
        check_interval             5
}

define service {
        use	                   generic-service
        hostgroup_name		   synology
        service_description	   UPS
        check_command		   synology_check!ups
        notifications_enabled      1
        check_interval             5
}
```
