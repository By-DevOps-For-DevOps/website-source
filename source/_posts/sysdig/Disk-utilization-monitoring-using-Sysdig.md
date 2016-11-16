---
title: Disk utilization monitoring using Sysdig
date: 2016-11-12 15:52:00
tags:
categories: sysdig
---

Disk utilization monitoring using Sysdig


### Creating a Dashboard tab

1. Under Explore tab select Server -> Overview.
2. Choose Group by host (host.mac).
3. On Table columns configuration (gear icon) Select the following fields
 - `fs.used.percent` - FS Usage %
 - `fs.root.used.percent` - FS Root Usage %
 - `fs.largest.used.percent`  - FS Largest Usage %
 - `fs.bytes.total` - FS Size
 - `fs.bytes.free` - FS Free Space
 - `fs.bytes.used` - Disk Used Bytes
4. Change the color coding by clicking on the gear icon of each of the columns. (default is yellow after 50% and red after 80%)
5. Pin the tab to the dashboard.

![creating a dashboard tab](/images/sysdig/Disk-utilization-monitoring-using-Sysdig-dashboard-tab.png)


### Creating an alert for 60% disk utilization

The following steps will help you to create an alert when the root directory (/) exceeds 60% of its usage:

1. Under alert tab click add alert button
2. Select the scope as `agent.tag.dcosName` or `region`. `agent.tag.dcosName` Is the tag we have added for the dcos cluster. `region` specify the aws region name. Assign the value for scope from the dropdown accordingly which one you selected as the scope.
3. Under `Set the condition` choose type as `manual`
4. For `Alert when` option Choose `fs.used.percent` as the metric > 60% as the threshold value.
5. For `Segment by` choose second option, select `Any of` and `fs.mountDir` metric.
![creating an alert for disk utilization](/images/sysdig/Creating-alert-for-disk-utilization.png)
6. Check the `Where` option and select `fs.mountDir` from the dropdown. Assign the value `/`
![creating an alert for disk](/images/sysdig/Creating-alert-for-disk-utilization2.png)
7. Choose the minimum monitor value as `1 min`.
8. Specify the Name, Description and Severity of the alert.
9. Enable the notification channel.
10. Enable automatic sysdig capture if necessary.
11. Click Create button.

