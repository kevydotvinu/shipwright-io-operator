# OLM Development

The Shipwright operator is meant to be deployed on a cluster using the
[Operator Lifecycle Manager](https://olm.operatorframework.io/) (OLM).
OLM provides mechanisms to support over the air upgrades and automatically deploy related operators
that are packaged in operator [catalogs](https://olm.operatorframework.io/).
Additional steps need to be taken to ensure the operator can be deployed with OLM.

## Prerequisites

* Ensure you have access to a Kubernetes cluster via `kubectl` with cluster admin permissions.
* Install Go version 1.17 or higher.
* Install OLM on your cluster. This can be done using the `make install-olm` command.
* Ability to push to a container registry that is accessible inside your Kubernetes cluster.

## Step 1: Build and push the operator and bundle

Run `make bundle-push IMAGE_REPO=<your-registry>`, pushing to a container registry that is
accessible inside your Kubernetes cluster.
This make command will push the operator and [operator bundle](https://olm.operatorframework.io/docs/tasks/creating-operator-bundle/)
to the container registry.
An operator bundle is an OCI artifact that tells OLM how to deploy your operator.

Using `ko.local` or `kind.local` for `IMAGE_REPO` is not recommended, as this will not push the
resulting images to an OCI-compliant container registry.
If you are using [KinD](https://kind.sigs.k8s.io/), follow the instructions on how to configure a
[local registry](https://kind.sigs.k8s.io/docs/user/local-registry/).
This will let you use `localhost:<port>` as your container registry.

## Step 2: Build and push an operator catalog

Next, run `make catalog-push IMAGE_REPO=<your-registry>`.
This will build and push an [operator catalog](https://olm.operatorframework.io/docs/tasks/creating-a-catalog/),
which packages your test operator bundle with the other operators available on [operatorhub.io](https://operatorhub.io).
The built catalog uses the new [file-based catalog](https://olm.operatorframework.io/docs/reference/file-based-catalogs/)
architecture, and packages the upstream Tekton operator along with the current released Shipwright
operator.
The catalog is rendered from JSON manifests in the `test/catalog` directory, and full contents of
the built catalog can be inspected in the `_output/catalog` directory.

As in step 1, be sure to use the same container registry for the `IMAGE_REPO` argument.

## Step 3: Deploy the operator using the catalog image

Finally, deploy the operator using `make catalog-run IMAGE_REPO=<your-registry>`, using the same
value for `IMAGE_REPO` as in the previous steps.
This will run a script that does the following:

1. Creates a custom [CatalogSource](https://olm.operatorframework.io/docs/tasks/make-catalog-available-on-cluster/)
   and [OperatorGroup](https://olm.operatorframework.io/docs/advanced-tasks/operator-scoping-with-operatorgroups/),
   which allows the operators in step 3's catalog to be installed anywhere on the cluster.
2. Creates a [Subscription](https://olm.operatorframework.io/docs/tasks/install-operator-with-olm/),
   which instructs OLM to install the operator and any dependent operators.
3. Checks that the operator has successfully been installed and rolled out.

Once the script completes, the Shipwright and Tekton operators will be installed on the cluster.

_Note:_

Scripts in `hack` folder may require `sed` (GNU), therefore in platforms other than Linux you may have it with a different name. For instance, on macOS it's usually named `gsed`, in this case provide the `SED_BIN` make variable with the alternative name.

```bash
$ make catalog-run SED_BIN=gsed ...
```
