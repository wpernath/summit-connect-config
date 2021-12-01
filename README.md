# summit-connect-config
This is the GitOps config repository for the summit connect demo

## Install from scratch
To install and use this demo from scratch, have an OpenShift 4.9 server available and install the following Kubernetes Operators from OperatorHub.io:

- OpenShift Pipelines
- OpenShift GitOps

Then create a summit-ci namespace via
```bash
$ oc new-project summit-ci
```

Then execute `pipelines.sh` from the `tekton` folder like:

```bash
$ ./pipeline.sh init \
    --git-user <your git user> \
    --git-password <your password> \
    --registry-user <your registry user> \
    --registry-password <your registry password>
```


