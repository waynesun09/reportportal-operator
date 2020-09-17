# Development

This operator is build with [operator-sdk](https://github.com/operator-framework/operator-sdk "operator-sdk") and ansible galaxy role [reportportal-openshift](https://github.com/waynesun09/reportportal-openshift "reportportal-openshift").

Note: Make sure you've installed the lastest operator-sdk as it changes rapidly.

Please read the [Operator SDK doc](https://sdk.operatorframework.io/docs/) to understand how the operator is built with the sdk.

For your devel enviroment, you could start with [CodeReady](https://github.com/code-ready/crc "CodeReady Containers") enviroment, which you have cluster admin role to deploy CRDs.

## Update Ansible Role

For update the ansible role, check and update in `roles/` .

Then follow the [Ansible user guide](https://sdk.operatorframework.io/docs/ansible/ "Ansible User Guide for Operator SDK") doc with build and test the operator.

After role updated, create new operator image with:
```console
$ buildah bud -f Dockerfile -t quay.io/waynesun09/rp5-operator:v0.0.5
$ buildah push quay.io/waynesun09/rp5-operator:v0.0.5
```

## Generate bundle file


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
$ sudo docker build -f bundle.Dockerfile -t quay.io/waynesun09/rp5-bundle-operator:v0.0.5 .
$ sudo operator-sdk bundle validate quay.io/waynesun09/rp5-bundle-operator:v0.0.5

$ sudo docker push quay.io/waynesun09/rp5-bundle-operator:v0.0.5
The push refers to repository [quay.io/waynesun09/rp5-bundle-operator]
4663510d6b4f: Pushed
61235e66d899: Pushed
35cd28174dce: Pushed
v0.0.5: digest: sha256:60985448712964fcb9ea2e22b82012b791c1647d27b18c34e1f7c5d376874188 size: 940
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
Note: Use sha256 rather than image tag to avoid cache problem

```console
$ sudo opm index add --container-tool docker --bundles quay.io/waynesun09/rp5-bundle-operator@sha256:60985448712964fcb9ea2e22b82012b791c1647d27b18c34e1f7c5d376874188 --from-index quay.io/waynesun09/wayne-index:1.0.3 --tag quay.io/waynesun09/wayne-index:1.0.4

$ sudo docker push quay.io/waynesun09/wayne-index:1.0.4
The push refers to repository [quay.io/waynesun09/wayne-index]
a3b7cbd3cadf: Pushed
dff05adcc153: Layer already exists
61fc2d1936e1: Layer already exists
4150c4f2e6df: Layer already exists
50644c29ef5a: Layer already exists
1.0.4: digest: sha256:240303dc7f4a904678948cc9c3f770ee21dbc9a4845a9815ed34f2002b5b252e size: 1371
```

Then could create CatalogSource on your testing cluster to add the operator registry.

Note: Use sha256 rather than image tag to avoid cache problem

## Run bundle scorecard

Check the `bundle/tests/scorecard/config.yaml` in the repo dir, update version and path if needed.

Run the corecard command, it'll test in your cluster env, so make sure you have connected to your cluster.
```console
$ operator-sdk scorecard bundle
```

## Generate packagemanifest

Default the operator is bundle format, if want to generate packagemanifest
```console
$ kustomize build config/manifests | operator-sdk generate packagemanifests -q --version 0.0.5

$ tree packagemanifests/
packagemanifests/
├── 0.0.5
│   ├── reportportal-operator.clusterserviceversion.yaml
│   ├── reportportal-operator-metrics-reader_rbac.authorization.k8s.io_v1beta1_clusterrole.yaml
│   ├── rp5.reportportal.io_reportportalrestores.yaml
│   ├── rp5.reportportal.io_reportportals.yaml
│   └── rp5.reportportal.io_serviceapiservicemonitors.yaml
└── reportportal-operator.package.yaml

1 directory, 6 files
```

As the default manifests base template is not fully supported with all the fields, so after generate simply replace the bundle CSV to the packagemanifests dir.
```console
$ cp bundle/manifests/reportportal-operator.clusterserviceversion.yaml packagemanifests/0.0.5/
```
