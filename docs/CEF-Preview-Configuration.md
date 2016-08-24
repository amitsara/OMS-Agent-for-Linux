# Configuration for Common Event Format (CEF) event collection (Preview)

##Configuration summary
1. Install and onboard the OMS Agent for Linux
2. Send the required events to the agent on UDP port 25225
3. Place the agent configuration [file](https://github.com/Microsoft/OMS-Agent-for-Linux/blob/4e90f23e62e935c32a90974ddc082b4966f26254/installer/conf/omsagent.d/security_events.conf) on the agent machine in ```/etc/opt/microsoft/omsagent/conf/omsagent.d/```
4. Restart the syslog daemon and the OMS agent


##Detailed configuration
####1. Download the OMS Agent for Linux, version 1.1.0-239 or above
  - [OMS Agent For Linux, Public Preview (2016-07)](https://github.com/Microsoft/OMS-Agent-for-Linux/releases/tag/v1.1.0-239)    

####2. Install and onboard the agent to your workspace as described here:
  - [Documentation for OMS Agent for Linux](https://github.com/Microsoft/OMS-Agent-for-Linux)  
  - [Syslog collection in Operations Management Suite](https://blogs.technet.microsoft.com/msoms/2016/05/12/syslog-collection-in-operations-management-suite/)  

####3.Send the required events to the OMS Agent for Linux
  1. Typically the agent is installed on a different machine from the one on which the events are generated.
	Forwarding the events to the agent machine will usually require several steps:
	- Configure the logging product/machine to forward the required events to the syslog daemon (e.g. rsyslog or syslog-ng) on the agent machine.
	- Enable the syslog daemon on the agent machine to receive messages from a remote system.
	
  2. On the agent machine, the events need to be sent from the syslog daemon to a local port the agent is listening on (by default: UDP port 25225).  
	*The following is an example configuration for forwarding all events from the local4 facility. 
	You can modify the configuration to fit your local settings.* 
	
	  **If the agent machine has an rsyslog daemon:**  
	  In directory ```/etc/rsyslog.d/```, create new file ```security-config-omsagent.conf``` with the following content:
	```
	#OMS_facility = local4
	local4.debug       @127.0.0.1:25225
	```  
	
	
	  **If the agent machine has a syslog-ng daemon:**  
	  In directory ``` /etc/syslog-ng/```, create new file ```security-config-omsagent.conf``` with the following content:
	```
	#OMS_facility = local4
	filter f_local4_oms { facility(local4); };
	destination security_oms { tcp("127.0.0.1" port(25225)); };
	log { source(src); filter(f_local4_oms); destination(security_oms); };
	```

####4. Place the following configuration file on the OMS Agent machine:  
  - [security_events.conf](https://github.com/Microsoft/OMS-Agent-for-Linux/blob/4e90f23e62e935c32a90974ddc082b4966f26254/installer/conf/omsagent.d/security_events.conf)  
  _Fluentd configuration file to enable collection and parsing of the events_  
	Destination path on Agent machine: ```/etc/opt/microsoft/omsagent/conf/omsagent.d/```  


####5. Restart the syslog daemon:  
```sudo service rsyslog restart``` or ```/etc/init.d/syslog-ng restart```


####6. Restart the OMS Agent:  
```sudo service omsagent restart``` or ```systemctl restart omsagent```


7. Confirm that there are no errors in the OMS Agent log:  
```tail /var/opt/microsoft/omsagent/log/omsagent.log```

8. The events will appear in OMS under the **CommonSecurityLog** type.  
Log search query: ```Type=CommonSecurityLog```
