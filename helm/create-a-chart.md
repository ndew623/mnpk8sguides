# Create a Chart

Let's start with a simple nginx deployment and see how to convert that into a Helm chart.

1. Start with this Deployment, call it `nginx-deployment.yaml`:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
```

2. Now create a new Helm chart named "my-nginx-chart":
```
$ helm create my-nginx-chart
Creating my-nginx-chart
$ ls
my-nginx-chart  nginx-deployment.yaml
```

3. The new chart gets filled with placeholder/example stuff. For this example, we want to clear all that out:
```
$ rm -r my-nginx-chart/templates/*
$ echo "" > my-nginx-chart/values.yaml
```

4. Move the deployment yaml into the `templates/` directory in the chart:
```
$ mv nginx-deployment.yaml my-nginx-chart/templates/
```

5. Finally, run `helm install`. It will install and create a single nginx Pod. You can see this with `kubectl get deploy` or `kubectl get pod`.
```
helm install nginx ./my-nginx-chart/
````

This is about as simple of a chart as you can get. The `values.yaml` file is empty, and the only template in `templates/` is an nginx Deployment with a single replica.

With our chart in the bare state, however, there is no way to use `values.yaml` to modify the installation.


## Using Values

To make the `values.yaml` file useful, we have to modify `my-nginx-chart/templates/nginx-deployment.yaml`.

Change `replicas: 1` to look like this instead:
```
  replicas: {{.Values.numReplicas}}
```

Instead of setting the value of replicas directly `{{.Values.numReplicas}}` will be substituded out for a value from `values.yaml`.

Next, `my-nginx-chart/values.yaml` should look like this:
```
$ cat my-nginx-chart/values.yaml 
numReplicas: 3
```

The name `numReplicas` is arbitrary. It could be `replicas`, `replicaCount`, or something else. It just has to match between the template and `values.yaml`.

Install again (after first uninstalling with `helm uninstall nginx`):
```
helm install nginx ./my-nginx-chart/
```
You should be able to use `kubectl get pod` or `kubectl get deploy` to see that 3 nginx Pods were created.

## More on values.yaml
Helm can do a lot more with templates and `values.yaml` than simple substitution.

To get an idea of what templating features Helm offers, you can look at the [Chart Template Guide](https://helm.sh/docs/chart_template_guide/getting_started/) from the Helm documenation.

You can also look at the chart we got in [Customize a Chart](./customize-a-chart.md) when running
```
helm pull oci://registry-1.docker.io/bitnamicharts/nginx
```
You can look at the deployment and service in the `templates/` directory of that chart to see a more complex example.