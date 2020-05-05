# Deploy with OLM via CLI

With OpenShift 4.x and CodeReady Containers the Operator Lifecycle Mangement Operator is installed by default.
Check with [OLM install doc](https://github.com/operator-framework/operator-lifecycle-manager/blob/master/doc/install/install.md "OLM install") for install OLM operator in your cluster if it's not installed.

The OLM operator application package have been pushed to: https://quay.io/application/waynesun09/reportportal-operator

## Create the OperatorSource

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
## View Available Operators
Once the `OperatorSource` and `CatalogSource` are deployed, the following command can be used to list the available operators (until an operator is pushed into quay, this list will be empty):
> The command below assumes `wayne-operators` as the name of the `OperatorSource` object. Adjust accordingly.
```console
$ oc get opsrc wayne-operators -o=custom-columns=NAME:.metadata.name,PACKAGES:.status.packages -n openshift-marketplace
NAME                PACKAGES
wayne-operators     reportportal-operator
```

## Create an OperatorGroup
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

## Create a Subscription
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

## Verify Operator health
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

## Create the Operand Custom Resource

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
