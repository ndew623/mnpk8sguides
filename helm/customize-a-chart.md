# Customizing a Helm Chart

Helm charts come with a lot of default settings for things like replica counts, namespaces, default passwords, and more.

Having default settings is useful, but in real-world use you'll need to modify some them sometimes.

## values.yaml
Most customization of Helm charts revolves around a file called `values.yaml` in the chart itself.

In the [Install a Chart](./install-a-chart.md) guide, we saw this `helm install` command:
```
helm install my-release oci://registry-1.docker.io/bitnamicharts/nginx
```

Now, instead of `helm install`, do `helm pull` and extract the tgz it downloads:
```
$ helm pull oci://registry-1.docker.io/bitnamicharts/nginx
Pulled: registry-1.docker.io/bitnamicharts/nginx:18.2.6
Digest: sha256:77a8b6c1e8085be52a7777ee889e1c415d1afb64924fe0bbc36780b209382b3a
$ ls
nginx-18.2.6.tgz
$ tar -xf nginx-18.2.6.tgz
$ ls
nginx  nginx-18.2.6.tgz
```

Look in the `nginx/` folder from the extracted tgz. The files in there make up the Helm chart.
```
$ cd nginx
$ ls
Chart.lock  Chart.yaml  README.md  charts  templates  values.schema.json  values.yaml
```

If you look at `values.yaml` you'll find it's 1000+ lines of nginx options and comments explaining them.

The artifacthub page for this chart has more documentation for these options: https://artifacthub.io/packages/helm/bitnami/nginx.

### Changing a value directly
Let's look at an example of customizing the chart by modifying the `values.yaml` file directly.

By default, the service created by this helm chart is of type LoadBalancer, which is not compatible with the awetomaton coder kind cluster setup. Let's modify the chart's `values.yaml` to fix that.

1. Find the settings for the service in `values.yaml`. (Try searching for the text "service.type". It's at line 631 as of this writing.)
2. Modify the `type:` under `service:` to be `NodePort` instead of `LoadBalancer`.
3. Modify the `nodePorts:` section to look like:
```
  nodePorts:
    http: "30005"
    https: ""
```
4. From in the directory containing the chart, run:
```
helm install nginx-values-changed .
```

The Helm chart should install. After the pod starts up, you should be able to see the default "Welcome to nginx!" page at port 30005.

(Remember not to have another service on port 30005 when trying this.)

### Changing values without downloading the chart first

We've seen how to download the chart using `helm pull`, and we've directly modified the values.yaml file there.

But running `helm pull` is not necessary to modify a chart's values.

Instead you can run
```
helm install -f my-values.yaml <release name> <chart>
```
where `my-values.yaml` contains only the options you want to change from their defaults.

#### Example

---
If you ran the last example, do these steps first:
- Run `helm uninstall nginx-values-changed` to cleanup the cluster.
- You can `rm` or at least `cd` out of the nginx chart directory from before, it isn't needed for this.

---
1. Create file called `my-values.yaml`.
2. Put only the values you want to change in `my-values.yaml`
In this case it will look like:
```
$ cat my-values.yaml
service:
  type: NodePort
  nodePorts:
    http: "30005"
```
3. Run:
```
helm install -f my-values.yaml nginx-values-file oci://registry-1.docker.io/bitnamicharts/nginx
```

The result should be the same as when we modified the `values.yaml` file directly, with the default nginx page at port 30005.