# Kustomize and Helm Combined

## Kustomize vs Helm

Kustomize and Helm have some overlap in their functionality.

The can both dynamically modify manifests for different cluster environments. Helm uses templates and values.yaml files, while kustomize makes changes via its patches.

Helm:
- Can be used to package, distribute, and deploy Kubernetes applications.
- Customization implemented with templates and values.yaml files.
  - Templates can implement some logic and text processing.
  - Values.yaml files can only change things that the templates' author chose not to "hardcode".
Kustomize:
- No features for packaging or distrubuting anything.
- Customization implemented with patch files.
  - Can change any "hardcoded" value in the targeted manifest.


Which tool should be used to change settings for different environments depends on the needs of the project.

Both are preferable to keeping and maintaining many copies of slightly different manifests for all different environments.

There are some cases where it makes sense to combine Helm and Kustomize. For example, you could use Helm to deploy and manage an off-the-shelf application, but use Kustomize to modify it in a way the Helm chart doesn't allow through values.yaml.

Some new platform applications are deployed this way.

## Using a Helm Chart with Kustomize

You should know that there is more than one way to combine Helm and Kustomize.

This example will use one way that is similar to how some apps in new platform do it.

### Example

This the example problem:
- Deploy an nginx helm chart to both a dev and a prod environment.
- Any Deployment from the chart should have a label "environment: prod" in the prod environment and "environment: dev" in the dev environment.

---

Start with a directory structure for kustomize:
```
.
├── base
│   └── kustomization.yaml
├── envs
│   ├── dev
│   │   ├── kustomization.yaml
│   │   └── label-patch.yaml
│   └── prod
│       ├── kustomization.yaml
│       └── label-patch.yaml
```

#### Base folder

If you read the [kustomize guide](../kustomize/README.md), you saw a `base/kustomization.yaml` that simply pointed to the "base" manifests like:
```
resources:
- some-manifest.yaml
- some-other-manifest.yaml
```

In our case, we don't have any manifests in our base folder, because we want to pull a helm chart.

So our `base/kustomization.yaml` file should look something like this:
```
helmCharts:
- releaseName: my-nginx
  name: nginx
  repo: https://some-repo-url
  version: x.x.x
```

This kustomization file specifies a helm chart, and a repo/chart name to pull. As you can see this is an example with a fake url and version. If you want to follow along with this example and see it work, I recommend copying the Helm chart from the [Helm guide](../helm/create-a-chart.md) into a `base/charts-dir/nginx` folder and changing `kustomization.yaml` to look like this:
```
helmGlobals:
  chartHome: charts-dir
helmCharts:
- name: nginx
  valuesFile: values.yaml
  releaseName: kustomize-helm-nginx
```
The folder structure would then look like:
```
├── base
│   ├── charts-dir
│   │   └── nginx
│   │       ├── charts
│   │       ├── Chart.yaml
│   │       ├── templates
│   │       │   └── nginx-deployment.yaml
│   │       └── values.yaml
│   └── kustomization.yaml
├── envs
│   ├── dev
│   │   ├── kustomization.yaml
│   │   └── label-patch.yaml
│   └── prod
│       ├── kustomization.yaml
│       └── label-patch.yaml
```

For this example, this setup lets us use a helm chart from a local server instead of a remote repo.

If you run:
```
$ cd base
$ kubectl kustomize --enable-helm
```
You should see kustomize spit out the complete helm template with no additional patches to it.

**Note:** You need the `--enable-helm` option when using kustomize with Helm.

#### Prod folder

In `envs/prod/kustomization.yaml`:
```
resources:
- ../../base
patches:
- path: label-patch.yaml
  target:
    kind: Deployment
```
This file:
- specifies the base `kustomization.yaml`.
- path(s) to a patch file(s)
- target selector that will apply to any Deployment

In `envs/prod/label-patch.yaml`:
```
- op: add
  path: "/metadata/labels/environment"
  value: prod
```

This patch file looks different from the previous example in the [kustomize guide](../kustomize/README.md), but it allows us to add a new label to the Deployment's metadata.

If you run `kubectl kustomize --enable-helm` from inside the `envs/prod` directory, kustomize will show this:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
    environment: prod # <-- new label
  name: nginx
spec:
  replicas: 3
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

It looks like the Deployment in the Helm chart's `templates/` folder, but it has `environment: prod` added under `metadata/labels`.

#### Dev folder
`envs/dev` should look nearly identical to `envs/prod`, except that the `value: prod` line in `label-patch.yaml` should say `value: dev`.
