# What is Kustomize

Kustomize is locally installed tool.

It comes built-in to `kubectl`.

The `kubectl version` command will output your `kubectl` and Kubernetes server versions, but it should also include the kustomize version that's built in to `kubectl`.

The purpose of kustomize is to help in using one manifest file across multiple clusters.

# Hypothetical Example

Imagine you have this deployment:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: example
  name: example
spec:
  replicas: 1
  selector:
    matchLabels:
      app: example
  template:
    metadata:
      labels:
        app: example
    spec:
      containers:
      - image: server:5000/my-image:test
        name: example-container
        resources:
          limits:
            memory: "8Mi"
            cpu: "250m"
        env:
        - name: ENVIRONMENT_VARIABLE_NAME
          value: ev-value
```

This deployment creates one replica running the `server:5000/my-image:test` image.

But running a single instance of a test image is probably only what you want on a development/research cluster.

For this example, imagine you want the production cluster to run 5 replicas, and you want it to run version 4.3.2 of `server:5000/my-image` rather than "test".

You could have seperate manifests for the development cluster and for the production cluster.

But when you have more manifests, that are more complicated, and on more than two clusters, keeping all the different manifests for all the different clusters organized and up-to-date becomes a mess.

Kustomize lets you have one base manifest shared across all of your clusters. Each cluster has its own kustomization file that specifies just the options that need to be changed from the base manifest.

# How it Works

Kustomize needs a specific directory structure, like this:
```
.
├── base
│   ├── deploy.yaml
│   └── kustomization.yaml
└── envs
    ├── dev
    │   ├── kustomization.yaml
    │   └── patches.yaml
    └── prod
        ├── kustomization.yaml
        └── patches.yaml
```

Side note: sometimes the `base` directory is called `resources`. Sometimes the `envs` directory is called `overlays`.

Under the `base` folder, `deploy.yaml` can just be the Deployment from the example above.

For this example, `base/kustomization.yaml` is very simple. It just names the base manifests that we'll be using, and in this case there is only one:
```
resources:
- deploy.yaml
```

You can probably tell that `envs/dev` and `envs/prod` will contain the cluster-specific settings. You could have many different environments under envs. You could also create more subdirectories, useful if you wanted to have more than one configuration for a single cluster.

`envs/dev/kustomization.yaml` and `envs/prod/kustomization.yaml` will look the same:
```
bases:
- ../../base
patches:
- path: patches.yaml
  target:
    kind: Deployment
    name: example
```

- `bases` points to the base directory
- `patches.yaml` ultimately contains the cluster specific settings
- `target` is used to specify what the settings in `patches.yaml` will be applied to. In this example, we only have a single manifest in bases, but there could be many manifests in a real-use scenario.

`envs/prod/patches.yaml` will look like this:
```
kind: Deployment
metadata:
  name: example
spec:
  replicas: 7
  template:
    spec:
      containers:
      - image: my-image:prod
        name: example-container
```

The `patches.yaml` file contains a spec for a Deployment, but only for the options that we are replacing. In this case, the container image and number of replicas.

The resource limits and environment variable from the Deployment are not included in this patches.yaml, so they will not be changed.

There a couple of quirks to `patches.yaml`:
1. `kind: Deployment` and `name: example` could be used to target this patch to our Deployment called example. The `target:` property in the `kustomization.yaml` overrides this.
  - So `kind:` and `name:` still need to be present, but because of `target:` their values don't matter.
    - really, with `target:` in the kustomization.yaml you could edit patches.yaml to have `kind: ASDF` and `name: qwerty` and it still works.
      - i don't fully understand why tbh
2. We didn't change the container name (`example-container`), but it still has to be included. This is because the `containers:` property is a list, and kustomize will use the `name` field from list elements to identify them.

# Running the Command
First, `cd envs/prod`.

Then run `kubectl kustomize`.

Kustomize will find the kustomization and patches files and output this:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: example
  name: example
spec:
  replicas: 7
  selector:
    matchLabels:
      app: example
  template:
    metadata:
      labels:
        app: example
    spec:
      containers:
      - env:
        - name: ENVIRONMENT_VARIABLE_NAME
          value: ev-value
        image: my-image:prod
        name: example-container
        resources:
          limits:
            cpu: 250m
            memory: 8Mi
```

That's the Deployment from the base folder, but with the replica and image settings from the patches.yaml.

# TODO how to use the output, kustomize applying to the cluster