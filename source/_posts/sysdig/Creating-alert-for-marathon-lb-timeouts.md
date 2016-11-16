---
title: Creating alert for marathon-lb timeouts
date: 2016-11-09 20:00:00
tags:
categories: sysdig
---

Creating alert for marathon-lb timeouts




Under Explore tab select Server -> Overview.
Choose Group by `marathon.app.name`.
Click on the bell button next to `marathon-lb`
A new alert popup will appear.
Under `Set the condition` choose type as `manual`
For `Alert when` option Choose `net.request.time.out`. Assign threshold value.
For `Segmented by` option choose second radio button and select `Any of` from dropdown. Select the filter metrics `proc.name` from the dropdown.
Check `Where` option and `proc.name` is `haproxy` .
Choose the minimum monitoring value as `1 min`.
Specify the Name, Description and Severity of the alert.
Enable the notification channel.
Enable automatic sysdig capture if necessary.
Click Create button.


![sysdig snapshot](/images/marathon-lb-timeouts.png)



