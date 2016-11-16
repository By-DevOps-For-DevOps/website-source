---
title: Monitoring CPU utilization using Sysdig
date: 2016-11-15 14:48:00
tags:
categories: sysdig
---

Monitoring CPU utilization using Sysdig



### Creating an alert for 90% CPU utilization

The following steps will help you to create an alert when a node exceeds 90% of its cpu utilization:
 
1. Under alert tab click add alert button.
2. Select the scope as `agent.tag.dcosName` or `region`. `agent.tag.dcosName` Is the tag we have added for the dcos cluster. `region` specify the aws region name. Assign the value for scope from the drop down accordingly which one you selected as the scope.
3. Under `Set the condition` choose type as `manual`
4. For `Alert when` option Choose `cpu.used.percent` as the metric > 90% as the threshold value.
5. For `Segment by` choose second option, select `Any of` and `host.hostName` metric.
6. Leave the `Where` option unchecked.
7. Choose the minimum monitor value as `5 min`.
8. Specify the Name, Description and Severity of the alert.
9. Enable the notification channel.
10. Enable automatic sysdig capture if necessary.
11. Click Create button.

![creating an alert for 90% CPU utilization](/images/sysdig/Creating-an-alert-for-90-CPU-utilization.png)