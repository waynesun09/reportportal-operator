# Deploy with CLI

OCP by default is installed with OLM, but you could deploy it directly with crd and create operands with coomand.

## Without OLM

User could deploy with crd and create CR instance.

    $ make install
    $ make deploy IMG=quay.io/waynesun09/rp5-operator:v0.0.6

    $ oc get crd |grep reportportal
    reportportalrestores.rp5.reportportal.io                    2020-09-17T13:08:36Z
    reportportals.rp5.reportportal.io                           2020-09-17T13:08:36Z
    serviceapiservicemonitors.rp5.reportportal.io               2020-09-17T13:08:36Z

    $ oc get clusterrole |grep reportportal
    reportportal-operator-metrics-reader                                   2020-09-17T12:50:44Z
    reportportal-operator.v0.0.6-8c9f6475f                                 2020-09-17T13:08:38Z
    reportportalrestores.rp5.reportportal.io-v1-admin                      2020-09-17T13:08:43Z
    reportportalrestores.rp5.reportportal.io-v1-crdview                    2020-09-17T13:08:43Z
    reportportalrestores.rp5.reportportal.io-v1-edit                       2020-09-17T13:08:43Z
    reportportalrestores.rp5.reportportal.io-v1-view                       2020-09-17T13:08:43Z
    reportportals.rp5.reportportal.io-v1-admin                             2020-09-17T13:08:43Z
    reportportals.rp5.reportportal.io-v1-crdview                           2020-09-17T13:08:43Z
    reportportals.rp5.reportportal.io-v1-edit                              2020-09-17T13:08:43Z
    reportportals.rp5.reportportal.io-v1-view                              2020-09-17T13:08:43Z
    serviceapiservicemonitors.rp5.reportportal.io-v1-admin                 2020-09-17T13:08:43Z
    serviceapiservicemonitors.rp5.reportportal.io-v1-crdview               2020-09-17T13:08:43Z
    serviceapiservicemonitors.rp5.reportportal.io-v1-edit                  2020-09-17T13:08:43Z
    serviceapiservicemonitors.rp5.reportportal.io-v1-view                  2020-09-17T13:08:43Z

    $ oc get rolebindings |grep reportportal
    reportportal-operator.v0.0.6-default-7c9bd8f688   Role/reportportal-operator.v0.0.6-default-7c9bd8f688   131m
    $ oc get clusterrolebindings |grep reportportal
    reportportal-operator.v0.0.6-8c9f6475f                                           ClusterRole/reportportal-operator.v0.0.6-8c9f6475f                                 131m

    $ oc get deploy |grep reportportal
    reportportal-operator-controller-manager   1/1     1            1           132m

Prepare and update the CR yaml file:

    $ cp config/samples/rp5_v1_reportportal.yaml cr_example.yaml

    Update the CR file with the parameters

    $ vim cr_example.yaml
    apiVersion: rp5.reportportal.io/v1
    kind: ReportPortal
    metadata:
      name: reportportal-sample
    spec:
      # You cluster default router hostname
      app_domain: apps.test-example.com
      ui_replicas: 1
      api_replicas: 1
      index_image: reportportal/service-index:5.0.10
      uat_image: reportportal/service-authorization:5.3.3
      ui_image: reportportal/service-ui:5.3.3
      api_image: quay.io/waynesun09/service-api:5.3.3-rootless
      migration_image: reportportal/migrations:5.3.3
      analyzer_image: reportportal/service-auto-analyzer:5.3.3
      index_replicas: 1
      gateway_replicas: 1
      enable_pg_restic_backup: 'yes'
      pg_restic_s3_bucket_init: 'yes'
      pg_s3_bucket_name: pgbackup-123123
      pg_restic_password: rp_user
      es_s3_backup_dir: s3_backup
      es_snapshot_bucket: es-snapshot-123123
      es_backup_schedule: '@daily'

With the parameters, you could input your cluster router host name for `app_domain` which will be used to create your app route name.
And also replicas for RP services components. All parameters descriptions could be in the `roles/reportportal/README.md`

Create new CR instance:

    $ oc create -f cr_example.yaml

    $ oc get ReportPortal
    NAME                   AGE
    reportportal-sample    1m

Check the deploy:

    $ oc get pods
    NAME                                                        READY   STATUS      RESTARTS   AGE
    analyzer-0                                                  1/1     Running     0          132m
    api-f6f95dcb8-klqgc                                         1/1     Running     6          125m
    elasticsearch-master-0                                      1/1     Running     0          133m
    elasticsearch-metrics-59b44b898f-8dmw7                      1/1     Running     0          132m
    gateway-5585458445-chhz2                                    1/1     Running     0          125m
    index-55cfc7696d-bq66n                                      1/1     Running     0          125m
    migrations-rmv6h                                            0/1     Completed   0          132m
    minio-0                                                     1/1     Running     0          133m
    postgresql-0                                                2/2     Running     1          120m
    rabbitmq-0                                                  1/1     Running     0          133m
    reportportal-operator-controller-manager-5b96bcb959-txskb   2/2     Running     0          135m
    uat-5d76864b4c-gs5kk                                        1/1     Running     5          125m
    ui-56588cf499-22bjb                                         1/1     Running     0          125m

    Check operator logs with progress
    $ oc logs reportportal-operator-controller-manager-5b96bcb959-txskb

If found any error or warning, deploy might have failed which need be addressed accordingly.

If you have specified the `app_domain` in your example, the ReportPortal instance should be available after all services have started and could be assess at:

    https://reportportal-{{ your_namespace }}.{{ app_domain }}

## Deploy with OLM

With OpenShift 4.x and CodeReady Containers the Operator Lifecycle Mangement Operator is installed by default.
Check with [OLM install doc](https://github.com/operator-framework/operator-lifecycle-manager/blob/master/doc/install/install.md "OLM install") for install OLM operator in your cluster if it's not installed.

The OLM operator application package have been pushed to: https://quay.io/application/waynesun09/reportportal-operator

## Create the CatalogSource

Prepare a catalog source yaml:

    $ cat catalog-source.yaml
    apiVersion: operators.coreos.com/v1
    kind: CatalogSource
    metadata:
      name: wayne-manifests
      namespace: openshift-marketplace
    spec:
      sourceType: grpc
      image: quay.io/waynesun09/wayne-index:1.0.4

The registry namespace is where the application is on quay.io.

Now add the source to the cluster:

    $ oc apply -f catalog-source.yaml

The `operator-marketplace` controller should successfully process this object:
```console
$ oc get catalogsource wayne-index -n openshift-marketplace
NAME              DISPLAY   TYPE   PUBLISHER   AGE
wayne-index                 grpc               9m48s
```

## Create an OperatorGroup
An `OperatorGroup` is used to denote which namespaces your Operator should be watching. It must exist in the namespace where your operator should be deployed, we'll use `rp` in this example.
Its configuration depends on whether your Operator supports watching its own namespace, a single namespace or all namespaces (as indicated by `spec.installModes` in the CSV).
Create the following file as  `operator-group.yaml` if your Operator supports watching its own or a single namespace.
If your Operator supports watching all namespaces you can leave the property `spec.targetNamespace` present but empty. This will create an `OperatorGroup` that instructs the Operator to watch all namespaces.
```yaml
apiVersion: operators.coreos.com/v1
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
apiVersion: operators.coreos.com/v1
kind: Subscription
metadata:
  name: rp-subscription
  namespace: rp
spec:
  channel: alpha
  name: reportportal-operator
  source: wayne-index
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
rp-subsription            reportportal-operator   wayne-index       alpha
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
$ cp config/samples/rp5_v1_reportportal.yaml cr-test.yaml
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
