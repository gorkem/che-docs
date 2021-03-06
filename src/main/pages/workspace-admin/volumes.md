---
title: "Volumes"
keywords: workspace, runtime, recipe, docker, stack, volume, volumes
tags: [workspace, runtime, docker, kubernetes]
sidebar: user_sidebar
permalink: volumes.html
folder: workspace-admin
---

{% include links.html %}

## Default Volumes

By default workspace containers/pods start with one default volume/PVC that persists `/projects` where workspace projects are physically located. When a workspace is stopped its machines are destroyed, however, volumes stay there.

## User-Provided Volumes

Your workspace may need additional volumes though, say, to persist a local Maven repo, node_modules, ruby jems, authorized_keys for ssh connections etc. You can add additional volumes for your workspace machines - each machine can get as many volumes as an underlying infrastructure can afford, or depending on imposed limits (that's mostly true for OpenShift).

Volumes can be added either in User Dashboard or directly in machine configuration:


```json
"volumes": {
  "myvolume": {
    "path": "/absolute/path/in/workspace"
  }
}
```
When adding a volume in User Dashboard using UI or config window, volume name and path are validated to avoid potential failures to create data volumes in Docker and PVCs in OpenShift. If you somehow update workspace configuration using REST API, there can be failures to attach volumes if:

* path containers `~` - an absolute path should be used
* name and path contains special characters, including `-` and `_`

If you want your workspace machines to share volumes, just create volumes for each machine with an identical name. This way, machines will share the same volume.
