# CronJob

- A CronJob creates Jobs on a repeating schedule.
- One CronJob object is like one line of a crontab (cron table) file. 
- It runs a job periodically on a given schedule, written in Cron format.
- CronJobs are useful for creating periodic and recurring tasks, like running backups or sending emails.
- CronJobs can also schedule individual tasks for a specific time, such as scheduling a Job for when your cluster is likely to be idle.

## Example
This example CronJob manifest prints the current time and a hello message every minute:

```
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```          