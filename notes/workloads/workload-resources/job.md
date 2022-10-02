# Jobs

- When you want to do a particular task though pod and then stop this will done through jobs
- We have replicaSets, daemonSets, statefulSets and deployments they all share pne common property:
they ensure that their pods are always running.
If the pods fails, the controller restarts it or reschedules it to another node to make sure the application pods is hosting keeps running

## Use cases

1. Take backup of a DB
2. Hel, charts uses jobs
3. running batch processes
4. run a task at ab schedule interval
5. logs rotation(e.g refresh logs after certain time)

> Job does not have to delete by itself, we have to delete it manually

```yaml
#job.yml
apiVersion: batch/v1
kind: Job
metadata:
  name: testjob
spec:
  template:
    metadata:
      name: testjob
    spec:
      containers:
      - name: counter
        image: centos:7
        command: ["bin/bash", "-c", "echo Technical-Guftgu; sleep 5"]
      restartPolicy: Never
---
apiVersion: batch/v1
kind: Job
metadata:
  name: testjob
spec:
  parallelism: 5             # Runs for pods in parallel
  activeDeadlineSeconds: 10  # Times out after 30 sec
  template:
    metadata:
      name: testjob
    spec:
      containers:
      - name: counter
        image: centos:7
        command: ["bin/bash", "-c", "echo Technical-Guftgu; sleep 20"]
      restartPolicy: Never

```

### The cron job pattern

- A CronJob creates Jobs on a repeating schedule.
- CronJobs are meant for performing regular scheduled actions such as backups, report generation, and so on. 
- If we have multiple nodes hosting the application for high availability, which node handles cron?
- what happens if multiple identical cron jobs run simultaneously?
  
```yaml
#cronjob.yml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
 name: bhupi
spec:
 schedule: "* * * * *"
 jobTemplate:
   spec:
     template:
       spec:
         containers:
         - image: ubuntu
           name: bhupi
           command: ["/bin/bash", "-c", "echo Technical-Guftgu; sleep 5"]
         restartPolicy: Never
```
