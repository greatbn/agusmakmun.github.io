---
layout: post
title: 'Tips: Force set time when chrony not work?'
date: '2019-05-25 11:33:07'
---

Time is very important in many systems such as database or storage.

If the gaps of time between two nodes in your cluster is too high, maybe your system is crashed.

There are many softwares using NTP(Network Time Protocol) to synchronize time for clients.

We are using Chrony, but sometimes It does not work as expected.

When you reboot a node, though chrony is started but the time still out of sync. I don't know this is a feature or bug of chrony. So we have to force chrony to sync with NTP Server.

But sometimes, It still does not work - My co-worker said.

So He suggests we should use and server and return the right format time of Linux. Then We will set the time manually.

Bellow is my script - Flask API then returns time for a client.



```
from flask import Flask
from datetime import datetime
from dateutil.tz import tzlocal



app = Flask(__name__)


@app.route("/time")
def get_time():
    format = "%a %b %d %H:%M:%S %Z %Y"
    return datetime.strftime(datetime.now(tzlocal()), format)


if __name__ == '__main__':
    app.run(host="0.0.0.0", port="30000")
```


You can put in your `rc.local` file this command to set correct time when boot


```
sudo date -s "`curl http://<IP of API>:30000/time`"
```

Actually, this is a workaround to set time when boot. If anyone has better solutions, Please leave a comment here. 