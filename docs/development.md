# Development

This operator is build with [operator-sdk](https://github.com/operator-framework/operator-sdk "operator-sdk") and ansible galaxy role [reportportal-openshift](https://github.com/waynesun09/reportportal-openshift "reportportal-openshift").

Note: Make sure you've installed the lastest operator-sdk as it changes rapidly.

Please read the [Operator SDK doc](https://sdk.operatorframework.io/docs/) to understand how the operator is built with the sdk.

For your devel enviroment, you could start with [CodeReady](https://github.com/code-ready/crc "CodeReady Containers") enviroment, which you have cluster admin role to deploy CRDs.

## Update Ansible Role

For update the ansible role, check and update in `roles/reportportal`.

Then follow the [Ansible user guide](https://sdk.operatorframework.io/docs/ansible/ "Ansible User Guide for Operator SDK") doc with build and test the operator.

## Update CSV

Update the CSV descriptions, spec and validations.

If need update CSV or update release version, follow guide [Generating a CSV](https://sdk.operatorframework.io/docs/olm-integration/generating-a-csv/) create the Operator bundle CSV package file.

Run operator-sdk bundle generate command
```console
$ operator-sdk generate csv --csv-version 0.0.1
```

## Upgrading your CSV
A new version of your CSV can be created by running:
```console
$ operator-sdk generate csv --csv-version <new-version>
```

## Build operator registry

Use `operator-registry` to build a registry image which include the operator application to quay.io

Check [operator-registry](https://github.com/operator-framework/operator-registry) to create Bundle images and Operators Index.

Then follow the guide with testing.

## Run scorecard

Check the .osdk-scorecard.yaml in the repo dir, update version and path if needed.

Run the corecard command, it'll test in your cluster env, so make sure you have connected to your cluster.
```console
$ operator-sdk scorecard
```
