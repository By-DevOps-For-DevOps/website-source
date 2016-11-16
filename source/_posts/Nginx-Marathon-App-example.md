---
title: Nginx Marathon App example
date: 2016-10-28 14:45:38
tags: Nginx Marathon App example
---
Here is an example how to run Nginx Marathon App in DCOS with [VIP](https://docs.mesosphere.com/1.8/usage/service-discovery/load-balancing-vips/virtual-ip-addresses/): 

```
{
  "id": "nginx",
  "cmd": null,
  "cpus": 1,
  "mem": 128,
  "disk": 0,
  "instances": 1,
  "container": {
    "docker": {
      "image": "nginx",
      "network": "BRIDGE",
      "portMappings": [
        {
          "containerPort": 80,
          "protocol": "tcp",
          "name": null,
          "labels": {
            "VIP_0": "1.1.1.4:8000"
          }
        }
      ]
    },
    "type": "DOCKER",
    "volumes": []
  },
  "env": {},
  "labels": {
    "HAPROXY_GROUP": "external"
  },
  "healthChecks": []
}
```
