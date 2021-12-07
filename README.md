# summit-connect-config
This is the GitOps config repository for the summit connect demo

## Install from scratch
To install and use this demo from scratch, have an OpenShift 4.9 server available and install the following Kubernetes Operators from OperatorHub.io:

- OpenShift Pipelines
- OpenShift GitOps

Then execute the following to create the summit-cicd namespace and everything else:
```bash
$ oc apply -f demo-setup/setup.yaml
```

This will create and initialize the following ArgoCD applications:
- summit-setup-apps: This will create the `argocd/summit-apps.yaml` ArgoCD applications
- summit-setup-tekton: This will create and initialize the `summit-cicd` namespace with the tekton pipelines.

After a while you should be able to get into the `summit-cicd` namespace.
Then execute `pipeline.sh` from the `tekton` folder to create the `Secret` and `ServiceAccount`s like : 

```bash
$ oc project summit-cicd
$ ./pipeline.sh init \
    --git-user <your git user> \
    --git-password <your password> \
    --registry-user <your registry user> \
    --registry-password <your registry password>
```

This installs everything necessary for executing the build and the stage pipelines including secrets etc.

Once everything was setup correctly, ArgoCD will automatically install the corresponding `summit-demo` in the correct namespace. 


## Workflow / Demo flow
### Using ArgoCD from OpenShift-GitOps
After you've installed the openshift-gitops operator, please go to the preconfigred ArgoCD instance:

```bash
$ oc get route openshift-gitops-server -ojsonpath='{.spec.host}' -n openshift-gitops
```

And use the admin password:

```bash
$ oc get secret openshift-gitops-cluster -n openshift-gitops -ojsonpath='{.data.admin\.password}' | base64 -d
```


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
$ oc apply -f demo-setup/setup.yaml
```

To install more or less the complete environment with DEV and STAGE apps running within a few seconds. 

## Uninstall everything
To uninstall everything, you simply need to delete the main two ArgoCD Applications by executing:

```bash
$ oc delete Application/summit-setup-apps -n openshift-gitops
$ oc delete Application/summit-setup-tekton -n openshift-gitops
```

## Note on using this demo with your own setup
Right now, everything is tied to use my repositories:
- quay.io/wpernath/summit-demo
- github.com/wpernath/summit-connect-config
- github.com/wpernath/summit-connect-quarkus-demo

If you want to use your own clones, you need to change the following files accordingly:
- `demo-setup/setup.yaml`: Both apps point to my github.com
- `argocd/summit-apps.yaml`: Both apps point to my github.com
- `tekton/intra/maven-settings-cm.yaml`: Points to my nexus repo in ci namespace
- `tekton/pipelines/dev-pipeline.yaml`: Default image repos and github sources point to my repos
- `tekton/pipelines/stage-release.yaml`: Default image repo and github config source point to my repos

To start the pipelines, you need set the default parameters accordingly. Or you're chaging the `tekton/pipelines.sh build|stage` parts of the script to pass the parameters on start.


** NOTE: You should be able to build _any_ default Quarkus application with this setup **

