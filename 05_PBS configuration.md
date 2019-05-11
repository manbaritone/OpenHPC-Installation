### Add holiday times to /var/spool/pbs/sched_priv/holidays
```
# vi /var/spool/pbs/sched_priv/holidays

*
YEAR  2019
*
* Prime/Nonprime Table 
*
*   Prime Non-Prime
* Day   Start Start
*
  weekday 0800  1730
  saturday  none  all
  sunday  none  all
*
* Day of  Calendar  Company
* Year    Date    Holiday
*

* if a holiday falls on a saturday, it is observed on the friday before
* if a holiday falls on a sunday, it is observed on the monday after

* Jan 1 
    1           Jan 1           New Year's Day
* Jan 2
    2           Jan 2           Substitution Day for New Year's Day
* Feb 19
   50           Feb 19          Makha Bucha Day
* Apr 8
   98           Apr 8           Substitution Day for Chakri Memorial Day
* Apr 15
  105           Apr 15          Songkran Festival Day
* May 20
  140           May 20          Substitution Day for Visakha Bucha Day
* Jul 16
  184           Jul 16          Asalha Bucha Day
* Jul 29
  201           Jul 29          Substitution Day for His Majesty the King's Birthday
* Aug 12
  224           Aug 12          Her Majesty the Queen Mother's Birthday
* Oct 14
  287           Oct 14          Substitution Day for King Bhumibol Adulyadej Memorial Day
* Oct 23
  296           Oct 23          Chulalongkorn Memorial Day
* Dec 5
  339           Dec 5           His Majesty the late King Bhumibol Adulyadej's Birthday
* Dec 25 
  359           Dec 25          Christmas Day
* Dec 26 
  360           Dec 26          Year-End Holiday
* Dec 27 
  361           Dec 27          Year-End Holiday
* Dec 30 
  364           Dec 30          Year-End Holiday
* Dec 31 
  365           Dec 31          Year-End Holiday
```

### Add GPU resource
```
# qmgr
create resource ngpus type=long, flag=nh
create resource gpu_id type=string, flag=h
exit
```

### Edit <sched_priv directory>/sched_config to add ngpus to the list of scheduling resources:
```
# vi /var/spool/pbs/sched_priv/sched_config 
resources: "ncpus, mem, arch, host, vnode, aoe, eoe, ngpus, gpu_id"
```

```
# service pbs restart 
```

### Add vnode for cpu and gpu execution
```
# qmgr
set node master resources_available.ncpus=0, resources_available.mem=0, resources_available.ngpus=0 
set node c1 resources_available.ncpus=0, resources_available.mem=0, resources_available.ngpus=0 
set node c2 resources_available.ncpus=0, resources_available.mem=0, resources_available.ngpus=0 
set node c3 resources_available.ncpus=0, resources_available.mem=0, resources_available.ngpus=0
exit
```

### Create vnode file for each vnode and scp to each host and ssh to each host
```
# ssh host
# pbs_mom -s insert vnodefile vnodefile
# service pbs restart
# exit
```

**vnodefile-cpu**
```
$configversion 2
master-cpu: Mom=master
master-cpu: resources_available.ncpus=8
master-cpu: resources_available.ngpus=0
master-cpu: resources_available.host=master
master-cpu: resources_available.vnode=master-cpu
master-cpu: sharing=default_shared
master-cpu: Priority=100
```

**vnodefile-gpu**
```
$configversion 2
master-gpu0: Mom=master
master-gpu0: resources_available.ncpus=1
master-gpu0: resources_available.ngpus=1
master-gpu0: resources_available.host=master
master-gpu0: resources_available.vnode=master-gpu0
master-gpu0: resources_available.gpu_id=gpu0
master-gpu0: sharing=default_excl
master-gpu0: Priority=10
```

```
# service pbs restart 
```

### Add compute node to PBS pro 
```
# qmgr
create node master
exit
# service pbs restart
```

### Add cpu queue
```
# qmgr
create queue qcpu 
set queue qcpu queue_type = Execution 
set queue qcpu resources_default.nodect = 1
set queue qcpu resources_default.nodes = 1:ppn=1
set queue qcpu resources_default.ncpus = 1
set queue qcpu resources_default.ngpus = 0
set queue qcpu enabled = True 
set queue qcpu started = True
exit
# service pbs restart
```

### Add gpu queue
```
# qmgr
create queue qgpu
set queue qgpu queue_type = Execution 
set queue qgpu resources_default.nodect = 1
set queue qgpu resources_default.nodes = 1:ppn=1
set queue qgpu resources_default.ngpus = 1
set queue qgpu resources_default.ncpus = 1
set queue qgpu enabled = True 
set queue qgpu started = True
exit
# service pbs restart
```
