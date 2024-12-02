# Intro/Installing a Chart

## What is Helm

Helm calls itself "the package manager for Kubernetes".

It's meant to do for Kubernetes what yum does for CentOS and pip does for python.

### Charts
A helm package is called a chart.

Charts can specify everything that their software needs to work.

For example, if you created a chart for Algorithm Bank, installing that chart would create and configure all of the Deployments and Services that a working AlgBank instance would need. Postgres, ab-orchestrator, ab-frontend, and everything else would all be created at once.

## Installing a Chart

Helm is a locally installed cli tool. It connects to, and manages the cluster by reading kubectl's `.kubeconfig` file.

After installing helm, to install a chart run this command:
```
helm install release-name <chart>
```
- `release-name` can be anything. It should be something that describes what you are deploying. Charts usually include this in the names of any resources they make (e.g. release-name-my-service and release-name-my-deployment).
- `<chart>` can be quite a few things. The documentation [here](https://helm.sh/docs/helm/helm_install/) goes into more detail, but the gist of it is that `<chart>` will be some kind of url or local path that leads to the chart you want to install.

### Install example
```
helm install my-release oci://registry-1.docker.io/bitnamicharts/nginx
```
After installation, you should be able to use `kubectl get deploy` and `kubectl get svc` to see it created a new Deployment and Service for nginx, both called `my-release-nginx`.

To see all of the charts you currently have installed from helm run:
```
$ helm list
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
my-release      default         1               2024-12-02 12:13:11.090363643 -0500 EST deployed        nginx-18.2.6    1.27.3
```

To uninstall the chart, run:
```
helm uninstall my-release
```
(`my-release` should match whatever name you chose in the `helm install` command.)

## Helm Repos

You can install charts directly, but you can also add repositories to Helm. Repositories are remote servers for hosting a collection of charts.

For example:
```
helm repo add bitnami https://charts.bitnami.com/bitnami
```

We can see the repo was added:
```
$ helm repo list
NAME                    URL
bitnami                 https://charts.bitnami.com/bitnami
```

We can see the charts it contains:
```
$ helm search repo bitnami
NAME                                            CHART VERSION   APP VERSION     DESCRIPTION
bitnami/airflow                                 22.3.1          2.10.3          Apache Airflow is a tool to express and execute...
bitnami/apache                                  11.2.22         2.4.62          Apache HTTP Server is an open-source HTTP serve...
bitnami/apisix                                  3.6.1           3.11.0          Apache APISIX is high-performance, real-time AP...
...
```

We can install a chart from the repo:
```
$ helm install my-release bitnami/nginx
```