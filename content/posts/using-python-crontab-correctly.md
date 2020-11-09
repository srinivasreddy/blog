+++ 
draft = false
date = 2019-12-09T11:46:07+05:30
title = "Using Python crontab correctly"
description = ""
slug = "" 
tags = ["python", "cron", "crontab"]
categories = []
externalLink = ""
series = []
+++

In this post, we are going to examine how to setup python-crontab correctly and debugging for possible errors and how to resolve them.

For the uninitiated, [python-crontab](https://gitlab.com/doctormo/python-crontab/) is a python package for editing cron table aka crontab used for scheduling the jobs run by cron.

I have setup a cron job which runs for an every hour. I have a file `every_hour.py` that contains the following code and sets up crontable entry to run `cron.py` for every hour.


```python
from crontab import CronTab
import os
import sys

if __name__ == "__main__":
    cron = CronTab(user=os.getlogin())
    cron.remove_all()
    cwd = os.getcwd()
    path = os.path.join(os.getcwd(), 'cron.py')
    output_file = os.path.join(os.getcwd(), 'crontab.output')
    os.chmod(path, 775)
    comment = "EveryHour"
    job = cron.new(command='cd {} && {} {} > {} 2>&1 '.format(cwd, sys.executable, path, output_file), comment=comment)
    job.hour.every(1)
    cron.write()
```


The following are the key takeaways while developing the solution.

1. Instead of hard coding the user argument, I get the logged-in user with `os.getlogin()`

2. I am writing only a single entry into cron table, so I want to make this operation idempotent.
hence `cron.remove_all()`. 

3. You can also use `crontab -l` to list the cron table entries and `crontab -r` to remove
all entries, and while `crontab -e` to edit the table entries manually. I recommend these commands
while diagnosing not to update a table entry. Because it is very easy to go wrong or introduce 
unnecessary line ending(s).

4. I am assigning the executing permissions to `cron.py` with `os.chmod(path, 775)`.

5. If you are in a virtual environment, your site-packages may not available to virtual environ python installation if you use `/usr/local/bin/python.` So it is always better to get the virtual env python
executable with the `sys.executable`.

6. `cd cwd` is perfectly fine because I want `cron` to go into that directory before executing the command.

7. `2>&1` and `output_file` help us diagnosing the output/error logs while cron running.

8. The command `job.every.hour(1)`  runs `cron.py` file every first minute of an hour.

9. While it is unrealistic to wait for an hour to check for cron, so change `job.every.hour(1)` to `job.every.minute(1)` for debugging.

10. And run `stat output_file` to see updated timestamps.


I hope this helps.









