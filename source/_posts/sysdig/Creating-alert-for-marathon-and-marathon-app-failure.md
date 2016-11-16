---
title: Creating alerts for marathon and marathon app failure
date: 2016-11-15 15:23:00
tags:
categories: sysdig
---

Creating alerts for marathon and marathon app failure

### Creating alerts for marathon failure

1. Under Explore tab select Server -> Overview.
2. Choose Group by `mesos.framework.name`.
3. Click on the bell button next to marathon framework ( marathon [<ip>:8080] ) from the list. A new alert popup will appear.
4. Under `Set the condition` choose type as `manual`
5. For `Alert when` option Choose `Entity is down`.
6. Leave `Where` option unchecked.
7. Choose the minimum monitoring value as `1 min`.
8. Specify the Name, Description and Severity of the alert.
9. Enable the notification channel.
10. Enable automatic sysdig capture if necessary.
11. Click Create button.


![Creating alerts for marathon failure](/images/sysdig/Creating-alerts-for-marathon-failure.png)

While grouping with `mesos.framework.name` on sysdig, it lists marathon from work running on one of the masters ( marathon [<ip>:8080] ). But the ip showing in the list might change when marathon is restarted. In that case we need to create alert for new listing name also. For solving this issue, we need to create alert for each of the master nodes. 
In order to do that:
- Go to Alerts page and select the alert we have created from above steps. Click copy for creating a similar alert. 
- Modify the master ip from the scope manually. 
- Specify the ip of another master and create a new alert. 
- Similarly create alert for each of the master nodes.

![Set the scope of nodes to watch](/images/sysdig/Creating-alerts-for-marathon-failure2.png)


### Creating alerts for marathon app failure
1. Under Explore tab select Server -> Overview.
2. Choose Group by `agent.tag.dcosName`.
3. Click on the bell button next to your dcos name 

    a. A new alert popup will appear.

4. Under `Set the condition` choose type as `manual`
5. For `Alert when` option Choose `Entity is down`.
6. For `Segmented by` option choose second radio button and select `Any of` from dropdown. Select the filter metrics `marathon.app.name` from the dropdown.
7. Leave `Where` option unchecked.
8. Choose the minimum monitoring value as `1 min`.
9. Specify the Name, Description and Severity of the alert.
10. Enable the notification channel.
11. Enable automatic sysdig capture if necessary.
12. Click Create button.

![Creating alerts for marathon app failure](/images/sysdig/Creating-alerts-for-marathon-app-failure.png)


