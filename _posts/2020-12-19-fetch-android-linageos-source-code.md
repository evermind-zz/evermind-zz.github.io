---
layout: post
title:  "Fetch shallow copy of Android LinageOs Sourcecode"
author: evermind
categories: [ android, lineageos, rom ]
---
```bash
#!/bin/bash
# if you do not want to fetch everything with history just the current state:
# --depth=1 ergibt nur eine shallow copy ohne history!
# the parameter -c for repo sync instructs it only to check out the branch nothing more.

repo init -u https://github.com/LineageOS/android.git -b cm-14.1 --depth=1
repo sync -c -n -j 4 
repo sync -c -l -j 16 # checkout from the synced stuff
```


### sources
[https://superuser.com/questions/603547/how-can-i-limit-the-size-of-the-android-source-i-need-to-download-with-repo-syn](https://superuser.com/questions/603547/how-can-i-limit-the-size-of-the-android-source-i-need-to-download-with-repo-syn)
