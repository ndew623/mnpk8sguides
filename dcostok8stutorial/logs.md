# Container Logs

You can get logs for containers from their Pods.

To see all of the Pods in the cluster run:
`kubectl get pod`

To get the logs for a Pod run:
`kubectl logs name-of-my-pod`

Kubernetes does not separate `stdout` and `stderr`.

## Some Common Options for kubectl logs

- You can get a continuous stream of logs to your terminal window with `-f` or `--follow`
  - `kubectl logs -f name-of-my-pod`
- You can limit how many lines are returned with `--tail=`
  - `kubectl logs --tail=20 name-of-my-pod` for 20 lines
- You can limit the amount of output by time with `--since=`
  - `kubectl logs --since=1h name-of-my-pod` for the last hour of logs
- You can print a timestamp with each line with `--timestamps`
  - `kubectl logs --timestamps name-of-my-pod`
- If you name a job in the `kubectl logs` command and prepend `job/` to the name, it will get logs from the most recent container for that job
  - `kubectl logs job/my-job-name`