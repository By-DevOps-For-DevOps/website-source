---
title: Troubleshooting common scenarios in DCOS
date: 2016-10-29 15:50:49
tags: Troubleshooting common scenarios in DCOS
categories: Troubleshooting
---

This doc contains information on how to troubleshoot common scenarios.

### Disk space issue
The instance running dcos might get filled up and dcos can fail. 
This can lead to unhealthy agents which may not respond to the Mesos. 
Follow the steps below to troubleshoot the scenario.

If able to identify the source of high disk space, 
try to do the cleanup and bring back the disk space. 
If there is a failure/bad configuration with docker, 
the `/var/lib/docker/overlay` folder can get used up. 
1. If able to identify the docker we can stop the container and it is not advised to delete the instance.
2. Disk space used by jenkins containers. 
We have had scenarios where the jenkins backup getting accumulated and using up the space. 
Stopping the backup plugin and deleting the backup from the container can resolve the issue.
3. If the above scenarios and scenario does not help, 
we can stop the instance from the console and wait for the auto scaling configuration to bring up new agents.

### IPTables blocking marathon-lb ports
This is where the `iptables` of the instances are setup to block the ports of marathon-lb and resulting timeouts.
You can use the below scripts to backup and flush the `iptables`.

```
for i in $(curl -sS leader-ip:5050/slaves | jq '.slaves[] | .hostname' | tr -d '"'); do ssh "core@$i" "mkdir -p debugging && sudo iptables-save >> debugging/iptables-backup && sudo iptables -F"; done
```

### Marathon service down
This scenario occurs when the marathon service running in one of the masters fails and does not restart on its own.

```
journalctl  -u dcos-marathon.service
systemctl try-restart dcos-marathon.service
```




