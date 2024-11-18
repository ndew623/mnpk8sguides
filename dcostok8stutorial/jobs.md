# Jobs

A Kubernetes job and a DC/OS job are similar.

A job is a Kubernetes resource.

Here is an example of a Job manifest:
```
apiVersion: batch/v1
kind: Job
metadata:
  name: hello-job
spec:
  template:
    spec:
      containers:
      - image: ubuntu
        name: hellojob
        command: ["echo", "HELLO!"]
      restartPolicy: Never
```
This Job simply prints "HELLO!" to the container's logs and exits.

In DC/OS, jobs could be container based or based on a command. In Kubernetes, there are only container based Jobs.

When Jobs run they create a Pod to run their container in. In [How to make a DC/OS-like service in K8s](./dcos-service-in-k8s.md) the Deployment manifest had a `template` section that was a template for all the Pods the Deployment might make. The Job manifest here is similar in that the `template` section contains options for the Pod.

When you `kubectl create` or `kubectl apply` the manifest above, the Job will start as soon as it is created.

### Re-running a Job
In short, you can run `kubectl delete -f my-job-manifest.yaml` and `kubectl create -f my-job-manifest.yaml` to delete and recreate the job. Upon recreation, it will run again.

There is also the `restartPolicy` option, set to `Never` in the above example. The two options for this are `Never` and `OnFailure`. With `OnFailure` the Job will continuously retry (with a new Pod each time) until it completes successfully (i.e. the container runs and returns an exit code of 0).

### Running a Job on a schedule
For this you need `kind: CronJob`.

For example:
```
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello-cron-job
spec:
  schedule: "*/2 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - image: ubuntu
            name: hellojob
            command: ["echo", "HELLO!"]
          restartPolicy: Never
```

It's essentially the same Job from before, but it is set to run every 2 minutes. The `schedule` option here uses the same cron format that DC/OS uses.

#### Concurrency
DC/OS jobs have a setting for concurrency.

Kubernetes CronJobs have a similar setting. Under `spec:` and next to `schedule:`, you can set `concurrencyPolicy:` to one of three things:
- Allow
  - the default
- Forbid
  - one at a time
  - must wait for the current job to finish
- Replace
  - I don't think this exists in DC/OS. It will cancel the running job and replace it with a new one.

## Cleaning up Pods
Finished Jobs and Pods don't go away on their own. They stick around in either a `Completed` or `Failed` state.

You can set `ttlSecondsAfterFinished` on a Job to delete after it has finished.
For example:
```
apiVersion: batch/v1
kind: Job
metadata:
  name: hello-job
spec:
  ttlSecondsAfterFinished: 180
  template:
    spec:
      containers:
      - image: ubuntu
        name: hellojob
        command: ["echo", "HELLO!"]
      restartPolicy: Never
```
It's the same Job from before, but it is automatically cleaned up 3 minutes after finishing.

CronJobs also have options for `successfulJobsHistoryLimit` (default: 3) and `failedJobsHistoryLimit` (default: 1) that will delete old Jobs.

When Jobs are deleted, any associated Pods are also deleted.