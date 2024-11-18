# Why YAML?

Essentially everything you can do with Kubernetes can be done by editing and applying YAML files.


But not everything *has* to be done with YAML.

## Doing things without YAML

The `kubectl` tool can do things directly without any YAML.

For example, here's a command that creates a `Deployment` without YAML:
```
$ kubectl create deploy --image=someimage:version --replicas=3 --port=8080 my-deployment-name
```

There are more `kubectl` commands for making Services, Jobs, and other things.

Kubernetes also has a web dashboard that can do some things via GUI interface.

## Why YAML is still better

The first reason is that not everything can be done directly with `kubectl` or the dashboard. Some more advanced configuration can only be done with YAML.

But it's more than that. Even if you just want a simple deployment and the command line options are sufficient, often you should still prefer YAML.

#### When the cluster is fully configured using YAML files:
- It can easily be recovered in the event that something (or everything) gets deleted. Simply re-apply all of the YAML files to get the exact same cluster.
- YAML configuration is self-documenting. You'll always know how the cluster is setup because it's described in plain text in the YAML file.
- The YAML files can be stored in git. With YAML files in git you can:
  - restore old configurations easily
  - create branches for alternate configurations of the cluster
  - view an exact history of changes to the cluster
    - this is great for debugging (and to find who to blame for breaking something)

#### If you make changes without using YAML:
- The configuration of the cluster will be opaque.
- It will be more difficult to identify and debug issues.
- It will be more difficult to recover a previous working cluster state.

# Terminology
Making changes using the dashboard or commands like `kubectl create deploy` is the **imperative** way of doing things.

Using YAML files and `kubectl apply` is the **declarative** way of doing things.

You sometimes see **declarative** and **imperative** come up in Kubernetes documentation and tutorials.



-----
(**Side note:** When learning Kubernetes, learning about YAML itself is worthwhile. It's mostly uncomplicated, but it has a few quirks.)