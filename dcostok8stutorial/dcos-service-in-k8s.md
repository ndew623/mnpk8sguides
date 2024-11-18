# How to make a DC/OS-like service in K8s

- I'm going to use DC/OS as a term to mean DC/OS+Mesos+whatever else makes the current platform work.
- In case you aren't familiar, K8s is a common abbreviation for Kubernetes.

1. [How to run a container in K8s](#how-to-run-a-container-in-k8s)
2. [DC/OS services vs K8s Services](#dcos-services-vs-k8s-services)

## How to run a container in K8s

### Overview
1. Create a manifest (i.e. a YAML file).
2. Put all of the settings/options you used to set in DC/OS in the YAML manifest.
3. `kubectl apply -f some-yaml-file.yaml`

### YAML File - More Detail

Here's an example YAML file:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-new-deployment
spec:
  replicas: 3 # <-- 3 instances
  selector:
    matchLabels:
      app: myapp
  template:
    metadata: labels:
        app: myapp
    spec:
      containers:
      - name: myappcontainer
        image: fakeserver:5000/someimage:v1.0 # <-- docker image to run
        ports:
        - containerPort: 8088 # <-- open port on 8088
        resources: # <-- resource limits
          limits:
            memory: "100Mi"
            cpu: "1"
          requests:
            memory: "100Mi"
            cpu: "1"
```

The important parts:
- The `kind` is `Deployment`. A K8s `Deployment` is kind of like a DC/OS service. `Deployments` have an image, multiple instances, volumes, resource limits, etc.
- In DC/OS you can have multiple instances of a service. For example, there is one ab-orchestrator but several team-manager instances. Instances in K8s are called `Pods`.
  - The `replicas: 3` setting above controls how many `Pods` the `Deployment` will make.
  - If you set the `replicas` to 3, the `Deployment` will create and maintain 3 `Pods`. If you set `replicas` to 1, the `Deployment` will make just 1 `Pod`.
- Other settings that are similar to DC/OS are `image`, `containerPort`, and cpu/memory limits.
  - This stuff is under `template` because it is a "template" to be used for each `replica`/`Pod` that the `Deployment` creates.

### kubectl - More Detail

The basic way to interact with a K8s cluster is using the kubectl command line tool.

#### kubectl - Common Commands

- `kubectl apply -f some-yaml-file.yaml`
  - Applies the manifest to the cluster. If you apply the manifest for a `Deployment` it will create that deployment. If there is already a `Deployment` with the same name, it will update it to match.
    - So for example, imagine you applied the YAML above to make a `Deployment`, but later you decided you wanted 6 `replicas`, not 3. Then you would edit the YAML file to say `replicas: 6` and run `kubectl apply` again.
- `kubectl create -f some-yaml-file.yaml`
  - `kubectl create` is just like `kubectl apply` except it can only create things, not edit existing things. It's kind of redundant.
- `kubectl get ...`
  - `kubectl get` is the simplest way to see what's currently in the cluster. To see `Deployments` run `kubectl get deployments`. To see `Pods` run `kubectl get pods`. To see `Jobs` run `kubectl get jobs`. There is also `kubectl get all` to see everything.
  - `kubectl` allows some abreviations. You can replace `pods` with `pod`, `jobs` with `job`, or `deployments` with either `deployment` or `deploy` in any command.
- `kubectl delete`
  - Removes things from the cluster either by type+name or by manifest.
  - For example, both of these commands would delete the `Deployment` from the example above:
    - `kubectl delete deploy my-new-deployment`
    - `kubectl delete -f some-yaml-file.yaml`

#### More kubectl
I'm not going to list every command, but here's some other things `kubectl` can do.
- Show container logs.
- Generate basic YAML manifests.
- Return the YAML manifest for anything currently in the cluster.
- Edit things in the cluster directly without applying a YAML file.
- Apply or delete all of the manifests in a directory rather than one file at a time.

## DC/OS services vs K8s Services
Both DC/OS and K8s have a thing called a service, but they do different things. Further up I wrote that a K8s Deployment was like a DC/OS service. It is in a lot of ways, but it's missing a major piece. On their own, Deployments aren't accessible. Deployments need to be paired with Services so that other things can reach them.

### Service YAML
Like everything in K8s, Services are created by a YAML manifest. For example:
```
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp # <-- identifies the Pods to point at by label
  ports:
  - protocol: TCP # <-- forwards port 9999 to port 8088
    port: 9999
    targetPort: 8088
  type: ClusterIP # <-- There are several service types, this one is for access from other Pods in the cluster
```
Important parts:
1. The Service's name is `myapp-service`. K8s has its own internal DNS based on Service names.
2. The `selector` controls what Pods the service points at. `myapp` matches the label given to the Pods of `my-new-deployment` in the Deployment YAML above.
3. Port 9999 is forwarded to port 8088. 8088 matches the port from `my-new-deployment`.
  - These 3 things together mean that this service will make `my-new-deployment` accessible in the cluster at `myapp-service:9999`.
    - Also valid are `myapp-service.default:9999` or `myapp-service.default.svc.cluster.local:9999`. In these longer URLs `default` is the namespace, and you'd need them to access the Service from another namespace.
- The type is `ClusterIP`, which is accessible to other things running in the cluster.
  - It's the default type of Service, so that line could've been omitted.

### Making Services accessible from outside the cluster
I put this section here because I think making Services externally accessible is the natural next thing to wonder about, but there isn't much I can write about it. That's because it depends on the configuration of cluster. There are many ways to setup external access to Services in the cluster.