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

Then execute `pipeline.sh` from the `tekton` folder like: 

```bash
$ ./pipeline.sh init \
    --git-user <your git user> \
    --git-password <your password> \
    --registry-user <your registry user> \
    --registry-password <your registry password>
```

This installs everything necessary for executing the build and the stage pipelines including secrets etc.

The final step is to install the demo resources. This can be done by executing 

```bash
$ oc apply -k argocd
```

This will create and initialize the ArgoCD applications including namespaces and roles for this. It will also install a PostgreSQL server in each namespace. 

Once everything was setup correctly, ArgoCD will automatically install the corresponding `summit-demo` in the correct namespace. 


## Workflow / Demo flow

### Building a new dev image
To build a new development version, please do your changes in the `summit-connect-quarkus-demo` Git repository and commit them. Then execute the following to start the `dev-pipeline`:

```bash
$ ./tekton/pipeline.sh build -u <your repo user> -p <your repo token>
```

This will build the sources and create a new image and pushes this image to quay.io. It finally updates the file `config/overlays/dev/kustomization.yaml` with the new image digest produced during build. 

ArgoCD will detect the changes and should update the `summit-dev` namespace accordingly.

### Building a staged release
To create a new release, execute the following to start the `stage-release` pipeline:

```bash
$ ./tekton/pipeline.sh stage -r v1.0.3
```

This will clone this Git repository, creates a new branch called `release-v1.0.3`, extracts the current image digest from `config/overlays/dev/kustomization.yaml`, tags that image on quay.io, updates the file `config/overlays/stage/kustomization.yaml` and pushes this change back to Git. 

The user must then create a pull request on GitHub and must merge this pull request back to `main`.

ArgoCD will detect the changes and should update `summit-stage` namespace accordingly.

## Sweet spots
IMHO, the biggest sweet spot of this demo is running 
```bash
$ oc apply -k argocd
```

To install more or less the complete environment with DEV and STAGE apps running within a few seconds. 