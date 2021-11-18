---
title: WORK
---

# WORK

## RRT
[NTAS_CI_FMS_SCC_AutoFinding_tracking](https://nokia.sharepoint.com/:x:/r/sites/NTAS_CI_FMS_SCC/_layouts/15/Doc.aspx?sourcedoc=%7B385E9FCA-C4D9-43A5-8E0A-3B4CDA140E07%7D&file=NTAS_CI_FMS_SCC_AutoFinding_tracking.xlsx&action=default&mobileredirect=true)
[EDA/RCA for collection branch](https://confluence.int.net.nokia.com/pages/viewpage.action?spaceKey=TASRnD&title=RRT+Failures+in+Collection+Branch+and+ECMS+Integration)




## mount logdrive
on linux server
```bash
mount -t cifs //eseesnrd90.nsn-rdnet.net/TASCI/TAS_EVO /root/store/logdrive -o user=manexec -o password=Testing2016q4 -o nobrl -o vers=3.0
```

on windows
```
```

## get floating ip in cloud instance
```
curl http://169.254.169.254/latest/meta-data/public-ipv4
```

# Central CI
```
# repoquery --location ntas-log-libs-1.16.0
https://repo.lab.pl.alcatel-lucent.com/ntas-yum-candidates/log-libs/ntas-log-libs-1.16.0-1.i686.rpm
https://repo.lab.pl.alcatel-lucent.com/ntas-yum-candidates/log-libs/ntas-log-libs-1.16.0-1.x86_64.rpm

pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple python-gitlab
```

