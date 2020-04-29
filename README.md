# Reportportal Operator

Umbrella Operator for ReportPortal and dependency services.

This repo is built with Operator SDK and Ansible role.

What services are included:

    - Report Portal v5.1
    - PostgreSQL
    - Elasticsearch
    - RabbitMQ
    - Minio

By default all services will be installed in the same namespace and managed by the operator.

## Requirements

- OpenShift Container Platform 4.x
- operator-sdk
- operator-courier
- oc client

## Deploy without OLM

User could deploy with crd and create CR instance.

Create the CRD:

    $ oc create -f deploy/crds/rp5.reportportal.io_reportportals_crd.yaml

    $ oc get crd |grep reportportal
    reportportals.rp5.reportportal.io                           2020-04-23T21:35:24Z

Deploy the Operator:

    $ oc create -f deploy/service_account.yaml
    $ oc create -f deploy/role.yaml
    $ oc create -f deploy/role_bindg.yaml
    $ oc create -f deploy/operator.yaml

Prepare and update the CR yaml file:

    $ cp deploy/crds/rp5.reportportal.io_v1alpha1_reportportal_cr.yaml cr_example.yaml

    Update the CR file with the parameters

    $ vim cr_example.yaml
    apiVersion: rp5.reportportal.io/v1alpha1
    kind: ReportPortal
    metadata:
      name: example-reportportal
    spec:
      # You cluster default router hostname
      app_domain:
      ui_replicas: 3
      api_replicas: 3
      index_replicas: 3
      gateway_replicas: 3

With the parameters, you could input your cluster router host name for app_domain which will be used to create your app route name.
And also replicas for RP services components. All parameters descriptions could be in the `roles/reportportal/README.md`

Create new CR instance:

    $ oc create -f cr_example.yaml

    $ oc get ReportPortal
    NAME                   AGE
    example-reportportal   1m


Check the deploy:

    $ oc get pods
    reportportal-operator-848ff6fdd-svfj4   2/2     Running     2          3m

    Check operator logs with progress
    $ oc logs reportportal-operator-848ff6fdd-svfj4 -c operator -f

If found any error or warning, deploy might have failed which need be addressed accordingly.

If you have specified the app_domain in your example, the ReportPortal instance should be available after all services have started and could be assess at:

    https://reportportal-{{ your_namespace }}.{{ app_domain }}

## Deploy with OLM via CLI

With OpenShift 4.x and CodeReady Containers the Operator Lifecycle Mangement Operator is installed by default.
Check with [OLM install doc](https://github.com/operator-framework/operator-lifecycle-manager/blob/master/doc/install/install.md "OLM install") for install OLM operator in your cluster if it's not installed.

The OLM operator application package have been pushed to: https://quay.io/application/waynesun09/reportportal-operator

### Create the OperatorSource

Prepare a operator source yaml:

    $ cat operator-source.yaml
    apiVersion: operators.coreos.com/v1
    kind: OperatorSource
    metadata:
      name: wayne-operators
      namespace: openshift-marketplace
    spec:
      type: appregistry
      endpoint: https://quay.io/cnr
      registryNamespace: waynesun09

The registry namespace is where the application is on quay.io.

Now add the source to the cluster:

    $ oc apply -f operator-source.yaml

The `operator-marketplace` controller should successfully process this object:
```console
$ oc get operatorsource wayne-operators -n openshift-marketplace
NAME                TYPE          ENDPOINT              REGISTRY   DISPLAYNAME  PUBLISHER   STATUS      MESSAGE                                       AGE
wayne-operators     appregistry   https://quay.io/cnr   waynesun09              Succeeded               The object has been successfully reconciled   30s
```
Additionally, a `CatalogSource` is created in the `openshift-marketplace` namespace:
```console
$ oc get catalogsource -n openshift-marketplace
NAME                           NAME        TYPE   PUBLISHER   AGE
wayne-operators                Custom      grpc   Custom      3m32s
[...]
```
### View Available Operators
Once the `OperatorSource` and `CatalogSource` are deployed, the following command can be used to list the available operators (until an operator is pushed into quay, this list will be empty):
> The command below assumes `wayne-operators` as the name of the `OperatorSource` object. Adjust accordingly.
```console
$ oc get opsrc wayne-operators -o=custom-columns=NAME:.metadata.name,PACKAGES:.status.packages -n openshift-marketplace
NAME                PACKAGES
wayne-operators     reportportal-operator
```

### Create an OperatorGroup
An `OperatorGroup` is used to denote which namespaces your Operator should be watching. It must exist in the namespace where your operator should be deployed, we'll use `rp` in this example.
Its configuration depends on whether your Operator supports watching its own namespace, a single namespace or all namespaces (as indicated by `spec.installModes` in the CSV).
Create the following file as  `operator-group.yaml` if your Operator supports watching its own or a single namespace.
If your Operator supports watching all namespaces you can leave the property `spec.targetNamespace` present but empty. This will create an `OperatorGroup` that instructs the Operator to watch all namespaces.
```yaml
apiVersion: operators.coreos.com/v1alpha2
kind: OperatorGroup
metadata:
  name: rp-operatorgroup
  namespace: rp
spec:
  targetNamespaces:
  - rp
```
Deploy the `OperatorGroup` resource:
```
$ oc apply -f operator-group.yaml
```

### Create a Subscription
The last piece ties together all of the previous steps. A `Subscription` is created to the operator. Save the following to a file named: `operator-subscription.yaml`:
```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: rp-operator-subsription
  namespace: rp
spec:
  channel: alpha
  name: reportportal-operator
  source: wayne-operators
  sourceNamespace: openshift-marketplace
```
In any case replace `<channel-name>` with the contents of `channel.name` in your `package.yaml` file.
Then create the `Subscription`:
```
$ oc apply -f operator-subscription.yaml
```

### Verify Operator health
Watch your Operator being deployed by OLM from the catalog source created by Operator Marketplace with the following command:
```console
$ oc get Subscription
NAME                      PACKAGE                 SOURCE            CHANNEL
rp-operator-subsription   reportportal-operator   wayne-operators   alpha
```
> The above command assumes you have created the `Subscription` in the `rp` namespace.
If your Operator deployment (CSV) shows a `Succeeded` in the `InstallPhase` status, your Operator is deployed successfully. If that's not the case check the `ClusterServiceVersion` objects status for details.
Optional also check your Operator's deployment:
```
$ oc get deployment -n rp
```

### Create the Custom Resource

Copy a CR yaml file
```console
$ cp deploy/crds/rp5.reportportal.io_v1alpha1_reportportal_cr.yaml cr-test.yaml
```

Update the app_domain with app route sub domain, e.g. apps-crc.testing.
Update replicas of the services.

Create the CR resource
```console
$ oc create -f cr-test.yaml
```

Check the application
```console
$ oc get all
```

## Deploy with OLM via GUI

Check the doc [Testing Operator Deployment on OpenShift](https://github.com/operator-framework/community-operators/blob/master/docs/testing-operators.md#testing-operator-deployment-on-openshift)

## Development

This operator is build with [operator-sdk](https://github.com/operator-framework/operator-sdk "operator-sdk") and ansible galaxy role [reportportal-openshift](https://github.com/waynesun09/reportportal-openshift "reportportal-openshift").

Note: Make sure you've installed the lastest operator-sdk as it changes rapidly.

Please read the Operator SDK doc to understand how the operator is built with the sdk.

For your devel enviroment, you could start with [CodeReady](https://github.com/code-ready/crc "CodeReady Containers") enviroment, which you have cluster admin role to deploy CRDs.

For update the ansible role, check and update in `roles/reportportal`.

Then follow the [Ansible user guide](https://github.com/operator-framework/operator-sdk/blob/master/doc/ansible/user-guide.md "Ansible User Guide for Operator SDK") doc with build and test the operator.

After the ansible operator is created, follow guide [Manage the operator using the Operator Lifecycle Manager](https://github.com/operator-framework/getting-started#manage-the-operator-using-the-operator-lifecycle-manager) create the Operator manifest. Which will generate the CSV package file.

### Generate CSV

Run operator-sdk generate command
```console
$ operator-sdk generate csv --csv-version 0.0.1 --update-crds
```

### Generate bundle file

Run operator-sdk bundle
```console
$ operator-sdk bundle create --generate-only
```

Then update the CSV descriptions, spec and validations.

### Package operator application

Use `operator-courier` to package and push the operator application to quay.io

Check [Pre-Requisites](https://github.com/operator-framework/community-operators/blob/772ef722f32042d1f865bdf9a74fe3339ed56b88/docs/testing-operators.md#pre-requisites) to check and push the application to quay.io

Then follow the guide with testing.

## CI

To be add
