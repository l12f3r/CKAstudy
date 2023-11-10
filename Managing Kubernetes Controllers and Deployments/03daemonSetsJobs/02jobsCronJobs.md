# Deploying and Maintaining Applications with DaemonSets and Jobs

## Jobs
- Ideal for running a single task on the cluster to completion
    - **Job** controller ensures that the specified number of Pods complete successfully
        - Checks the return code from the executed program to confirm if completion was successful (0)
- Creates one or more Pods
- Example workloads:
    - Ad-hoc that need to be executed periodically
    - Batch jobs
    - Data-oriented task (i.e. executing a backup, moving data around)

#### Job controller techniques to ensure completion
- Pod has an **interrupted execution** (if a node fails inside of the cluster, for instance)
    - Pod is rescheduled somewhere else in the cluster
- **Non-zero Exit Code** is returned
    - Follows the container `restartPolicy` instructions on how to react
        - Only `onFailure` or `Never` policy types must be used on Jobs

### Job lifecycle
- When a Job is successfully completed
    - Its status is set to `Completed`
    - The Job object remains
    - Pods are not deleted
        - Must be kept around for logs and other outputs
    - Job deletion after completing is arbitrary: if selected, its Pods are also deleted

### Job declaration
```
apiVersion: batch/v1
kind: Job                                           # this is where the Job is defined
metadata:
  name: hello-world
spec:
  backoffLimit: 3                                   # number of re-attempts to run Job if execution if failed
  completions: 50                                   # number of copies of this Pod to be executed to completion
  parallelism: 10                                   # maximum number of Pods running concurrently
  template:
    spec:
      containers:                                   # spec for containers to run this Job
      - name: ubuntu
        image: ubuntu
        command:                                    # this is how a command is passed to the instance to be provisioned
          - "/bin/bash"
          - "-c"
          - "/bin/echo Hello from Pod $(hostname) at $(data)"
      restartPolicy: Never                          # explicitly defined, because the default (Always) is not compatible
```

### Controlling Job execution
- Configuration attributes:
    - `backoffLimit`: number of Job attempts to recreate Pods before it's marked as failed (default: 6)
    - `activeDeadlineSeconds`: duration in seconds of the max execution time for the Job - it runs for this duration before the system tries to terminate it
    - `parallelism`: max number of desired Pods a Job should run at a point in time (good for breaking large chunks of data into computable tasks), concurrently
    - `completions`: number of Pods that need to finish successfully (ordinary Jobs usually have a `1` value; larger values must be defined to leverage parallelism on larger Jobs), in total

## CronJobs
- For running Jobs periodically, **CronJob** runs a Job on a given time-based schedule (similar to Linux's cron job)
    - It even uses the standard cron format
- Example workloads:
    - Periodic workloads and scheduled tasks (backups, data movements etc)
- A CronJob resource is created when the object is submitted to the API Server by the administrator
    - When the given time-based moment arrives, a Job is created via the Job template from the CronJob object

### Controlling CronJob execution
- Configuration attributes:
    - `schedule`: cron formatted schedule
    - `suspend`: allows to suspend subsequent executions of the CronJob
    - `startingDeadlineSeconds`: amount of time to mark the Job as failed if it hasn't started
    - `concurrencyPolicy`: handles concurrent executions of a Job (either Allow, Forbid or Replace)
    - `succesfulJobsHistoryLimit`: limit of successfully executed Jobs saved in the cluster

### CronJob declaration
```
apiVersion: batch/v1
kind: CronJob                            # where the CronJob is defined
metadata:
  name: hello-world-cron
spec:
  schedule: "*/1 * * * *"               # the cron-formatted schedule
  jobTemplate:                          # where the scheduled Job is defined
    spec:
      template:                         # Pod template
        spec:
          containers:                   # defining containers to run inside this Job
          - name: ubuntu
 ...
```

###### Return to [Summary](README.md)