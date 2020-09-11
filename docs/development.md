# Development

This operator is build with [operator-sdk](https://github.com/operator-framework/operator-sdk "operator-sdk") and ansible galaxy role [reportportal-openshift](https://github.com/waynesun09/reportportal-openshift "reportportal-openshift").

Note: Make sure you've installed the lastest operator-sdk as it changes rapidly.

Please read the [Operator SDK doc](https://sdk.operatorframework.io/docs/) to understand how the operator is built with the sdk.

For your devel enviroment, you could start with [CodeReady](https://github.com/code-ready/crc "CodeReady Containers") enviroment, which you have cluster admin role to deploy CRDs.

## Update Ansible Role

For update the ansible role, check and update in `roles/reportportal`.

Then follow the [Ansible user guide](https://sdk.operatorframework.io/docs/ansible/ "Ansible User Guide for Operator SDK") doc with build and test the operator.

After role updated, create new operator image with:
```console
$ buildah bud -f build/Dockerfile -t quay.io/waynesun09/rp5-operator:v0.0.5
$ buildah push quay.io/waynesun09/rp5-operator:v0.0.5
```

## Update CSV

If need update CSV or update release version, follow guide [Generating a CSV](https://sdk.operatorframework.io/docs/olm-integration/generating-a-csv/) create the Operator manifest CSV package file, make sure to update new operator image and related changes.

Run operator-sdk generate command
```console
$ operator-sdk generate csv --csv-version 0.0.3 --update-crds
```

## Generate bundle file

Run operator-sdk bundle
```console
$ operator-sdk bundle create --generate-only
```

Make sure the version in annotations are same (alpha, stable, etc.) are same in csv.

Then update the CSV descriptions, spec and validations.

## Validate the bundle file

```console
$ operator-sdk bundle validate deploy/olm-catalog/reportportal-operator
```

## Test the bundle
Check [Testing Operator Deployment with OLM](https://sdk.operatorframework.io/docs/olm-integration/olm-deployment/)

```console
$ operator-sdk run packagemanifests
```

## Create bundle image
The default bundle create image builder is docker, make sure you have installed latest docker on you host and docker daemon is running.

Podman did not support Docker v2.2 format yet, while v2.1 schema format is not supported by operator registry.

```console
$ sudo operator-sdk bundle create quay.io/waynesun09/rp5-bundle-operator:v0.0.5 --channels alpha
```

The channel must be same in the metadata/annotations.yaml

Push the bundle image to quay.io

```console
$ sudo docker push quay.io/waynesun09/rp5-bundle-operator:v0.0.5
```

## Build operator registry

Use `operator-registry` to build a registry index image which include the operator application to quay.io

Check [operator-registry](https://github.com/operator-framework/operator-registry) to create Bundle images and Operators Index.

```console
$ sudo opm index add --container-tool docker --bundles quay.io/waynesun09/rp5-bundle-operator:0.0.1 --tag quay.io/waynesun09/wayne-index:1.0.0
$ sudo docker push quay.io/waynesun09/wayne-index:1.0.0
```

To add updated bundle image to the index:

```console
$ sudo opm index add --container-tool docker --bundles quay.io/waynesun09/rp5-bundle-operator:v0.0.5 --from-index quay.io/waynesun09/wayne-index:1.0.3 --tag quay.io/waynesun09/wayne-index:1.0.4
$ sudo docker push quay.io/waynesun09/wayne-index:1.0.4
```

Then could create CatalogSource on your testing cluster to add the operator registry.

## Run scorecard

Check the .osdk-scorecard.yaml in the repo dir, update version and path if needed.

Run the corecard command, it'll test in your cluster env, so make sure you have connected to your cluster.
```console
$ operator-sdk scorecard
```
