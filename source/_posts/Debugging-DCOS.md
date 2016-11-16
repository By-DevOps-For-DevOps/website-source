---
title: Debugging DCOS
date: 2016-10-31 09:09:33
tags: Debugging DCOS
---
### Steps - To be discussed and updated

For troubleshooting, once we ssh into the machine create a folder in `/home/core/debugging` and make sure that the backups of the configurations to be kept there.

Before closing the ssh session `history >> /home/core/debugging/history` to keep a track of the steps you have performed to troubleshoot.

Once you back into the manager:
1. For easy troubleshooting install JQ from steps below.
```
wget http://stedolan.github.io/jq/download/linux64/jq 
chmod +x ./jq 
cp jq /usr/bin
```

2. To compress the debugging folder of each slaves.
```
for i in $(curl -sS 10.0.3.89:5050/slaves | jq '.slaves[] | .hostname' | tr -d '"'); do ssh "core@$i" "sudo tar -zcvf debugging-$i.tar.gz debugging"; done

mkdir debugging

for i in $(curl -sS 10.0.3.89:5050/slaves | jq '.slaves[] | .hostname' | tr -d '"'); do scp "core@$i:/home/core/debugging-$i.tar.gz" "debugging/"; done

tar -zvcf debugging.tar.gz debugging
```

Either we can scp the compressed file to our local machine or push it to an s3 bucket.





















