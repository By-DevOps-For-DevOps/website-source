---
title: Troubleshoot CoreOS in general which you will need in DCOS
date: 2016-10-30 09:07:29
tags: Troubleshoot CoreOS in DCOS
categories: Troubleshooting
---
These are supposed to be the steps followed to troubleshoot CoreOS in general.

`ssh` into the machine and check for initial login warnings.

`systemctl status`, this will list out the status of different services.

`systemctl list-units`, this will list the services with their status

`systemctl status name.service`, to find out the status of individual services.

`journalctl -u name.service`, to list out the logs of service.

You can use various journalctl parameter combination to fetch the logs of different time.


### Systemctl Cheatsheet

```
sudo systemctl start application.service
sudo systemctl stop application.service
sudo systemctl restart application.service
sudo systemctl reload application.service
sudo systemctl reload-or-restart application.service
sudo systemctl enable application.service
sudo systemctl disable application.service
sudo systemctl status application.service
sudo systemctl try-restart application.service
```

[Journalctl Cheatsheet](https://www.digitalocean.com/community/tutorials/how-to-use-journalctl-to-view-and-manipulate-systemd-logs)
