# Development

This operator is build with [operator-sdk](https://github.com/operator-framework/operator-sdk "operator-sdk") and ansible galaxy role [reportportal-openshift](https://github.com/waynesun09/reportportal-openshift "reportportal-openshift").

**Note:** Make sure you've installed the lastest operator-sdk as it changes rapidly.

Please read the [Operator SDK doc](https://sdk.operatorframework.io/docs/) to understand how the operator is built with the sdk.

For your devel enviroment, you could start with [CodeReady](https://github.com/code-ready/crc "CodeReady Containers") enviroment, which you have cluster admin role to deploy CRDs.

## Update Ansible Role

For update the ansible role, check and update in `roles/` .

Then follow the [Ansible user guide](https://sdk.operatorframework.io/docs/ansible/ "Ansible User Guide for Operator SDK") doc with build and test the operator.

After role updated, create new operator image with:
```console
$ buildah bud -f Dockerfile -t quay.io/waynesun09/rp5-operator:v0.0.7
$ buildah push quay.io/waynesun09/rp5-operator:v0.0.7
```

## Generate bundle file

Update Makefile with target ${VERSION}

Run make bundle
```console
$ make bundle
```
It will ask you to input interactively for some CSV fields.

Not all part could be generated so far, the CRD spec is the part you need check and update manually.

## Create bundle image and validate

The default bundle create image builder is docker, make sure you have installed latest docker on you host and docker daemon is running.

Podman did not support Docker v2.2 format yet, while v2.1 schema format is not supported by operator registry.


```console
$ sudo docker build -f bundle.Dockerfile -t quay.io/waynesun09/rp5-bundle-operator:v0.0.7 .

$ sudo docker push quay.io/waynesun09/rp5-bundle-operator:v0.0.7
The push refers to repository [quay.io/waynesun09/rp5-bundle-operator]
4663510d6b4f: Pushed
61235e66d899: Pushed
35cd28174dce: Pushed
v0.0.7: digest: sha256:2b2be124605a513738a59db74ef3bed651d5b08d2e7f32c4bdd4f6d252f9b6c3 size: 940

$ sudo operator-sdk bundle validate quay.io/waynesun09/rp5-bundle-operator:v0.0.7
```

The channel must be same in the bundle/metadata/annotations.yaml and the bundle.Dockerfile.

## Build operator registry

Use `operator-registry` to build a registry index image which include the operator application to quay.io

Check [operator-registry](https://github.com/operator-framework/operator-registry) to create Bundle images and Operators Index.

```console
$ sudo opm index add --container-tool docker --bundles quay.io/waynesun09/rp5-bundle-operator:0.0.1 --tag quay.io/waynesun09/wayne-index:1.0.0
$ sudo docker push quay.io/waynesun09/wayne-index:1.0.0
```

To add updated bundle image to the index:

Replace sha256 value of the bundle image at follow opm index command:

```console
$ sudo opm index add --container-tool docker --bundles quay.io/waynesun09/rp5-bundle-operator@sha256:c2aa752b16fd66ef66b3e955863a772b03d46c1cc4499c302f0fec1425f50ec4 --from-index quay.io/waynesun09/wayne-index:1.0.5 --tag quay.io/waynesun09/wayne-index:1.0.6
```

**Note:** Use sha256 rather than image tag to avoid cache problem

```console
$ sudo docker push quay.io/waynesun09/wayne-index:1.0.6
The push refers to repository [quay.io/waynesun09/wayne-index]
a3b7cbd3cadf: Pushed
dff05adcc153: Layer already exists
61fc2d1936e1: Layer already exists
4150c4f2e6df: Layer already exists
50644c29ef5a: Layer already exists
1.0.6: digest: sha256:5ef938574b293a04d25a19572534104ff92c1caa4ce07f40afbb002be856ed89 size: 1371
```

Then could create CatalogSource on your testing cluster to add the operator registry.

**Note:** Use sha256 rather than image tag to avoid cache problem

## Create CatalogSource

Check the doc [Using the index with Operator Lifecycle Manager](https://github.com/operator-framework/operator-registry#using-the-index-with-operator-lifecycle-manager)

The OLM operator bundle have been added to registry index image: https://quay.io/waynesun09/wayne-index in previous step.

### Create the CatalogSource

Prepare a catalog source yaml:

    $ cat catalog-source.yaml
    apiVersion: operators.coreos.com/v1
    kind: CatalogSource
    metadata:
      name: wayne-index
      namespace: openshift-marketplace
    spec:
      sourceType: grpc
      image: quay.io/waynesun09/wayne-index:1.0.6

In the OperatorHub choose openshift-marketplace namespace and select Provider type as `wayne-index` as defined in the catalog.

![alt text](docs/operatorhub-catalogsource.png "OperatorHub")

## Run bundle scorecard

Check the `bundle/tests/scorecard/config.yaml` in the repo dir, update version and path if needed.

Run the corecard command, it'll test in your cluster env, so make sure you have connected to your cluster.
```console
$ operator-sdk scorecard bundle
```

## Submit to Community Operators

Reportportal Operator switches from packagemanifest to bundle format start from v0.0.6, which make update the operator in Community Operator more easier as the operator is build with latest operator-sdk and bundle is the default format.

When new version of operator have been build, in the [reportportal-operator](https://github.com/operator-framework/community-operators/tree/master/community-operators/reportportal-operator) dir, create new version dir, e.g. 0.0.7, and copy bundle/manifests bundle/metadata into the new version dir. Also copy the bundle.Dockerfile and rename as Dockerfile and update COPY steps to:

       COPY manifests /manifests/
       COPY metadata /metadata/

**Note**: current bundle.Dockerfile is for build from project dir, while bundle build is from bundle dir, that's why need update the dir. Also the tests dir is deleted as for production.

Create new PR request with the new bundle version and fix CI and review issues.
