# synology-nagios-plugin
A simple Nagios Plugin which checks the health of a Synology NAS
It supports checking the following items:
* System status
* Power status
* Fan status
* Disks status
* RAID (Volumes) status
* DSM version and update status
* System temperature
* UPS information

## Based on
This plugin is based on plugin by deegan199. It can be found [here.](https://exchange.nagios.org/directory/Plugins/Network-and-Systems-Management/Others/Synology-status/details)

### Changes From Original
This version supports volume usage on DSM 6.2. Also, it checks readings by type,
rather then checking everything at once. This allows for splitting checks into
different services in Nagios. Look below for more information. As a part of this
change, the verbose option was removed and it is now always verbose. Also, the
UPS option was removed and added as a check type instead (but it hasn't been tested with a UPS).


## Requirements
snmpwalk and snmpget need to be installed. Also, the SNMP agent on the NAS has to be activated.

## Usage
```
usage: ./check_snmp_synology [OPTIONS] -u [user] -p [pass] -h [hostname]
options:
            -u [snmp username]   	Username for SNMPv3
            -p [snmp password]   	Password for SNMPv3

            -2 [community name]	  	Use SNMPv2 (no need user/password) & define community name (ex: public)

            -h [hostname or IP](:port)	Hostname or IP. You can also define a different port

            -W [warning temp]		Warning temperature (for disks & synology) (default 50)
            -C [critical temp]		Critical temperature (for disks & synology) (default 60)

            -w [warning %]		Warning storage usage percentage (default 80)
            -c [critical %]		Critical storage usage percentage (default 95)

            -t [check type]	        The type of check to perform, must be one of the following: system version temperature power fan disk raid ups

            -i   			Ignore DSM updates

examples:	./check_snmp_synology -u admin -p 1234 -h nas.intranet -t temperature
	     	./check_snmp_synology -u admin -p 1234 -h nas.intranet -v -t system
		./check_snmp_synology -2 public -h nas.intranet -t power
		./check_snmp_synology -2 public -h nas.intranet:10161 -t ups
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
