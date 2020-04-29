Reportportal Operator
=====================

Umbrella Operator for ReportPortal and dependency services.

This repo is built with Operator SDK and Ansible role.

What services are included:

    - Report Portal v5.1
    - PostgreSQL
    - Elasticsearch
    - RabbitMQ
    - Minio

By default all services will be installed in the same namespace and managed by the operator.

Requirements
------------

OpenShift Container Platform 4.x

Deploy without OLM
------------------

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

Deploy with OLM
---------------

With OpenShift 4.3 and CodeReady Containers the Operator Lifecycle Mangement Operator is installed by default.

Development
-----------

This operator is build with [operator-sdk](https://github.com/operator-framework/operator-sdk "operator-sdk") and ansible galaxy role [reportportal-openshift](https://github.com/waynesun09/reportportal-openshift "reportportal-openshift").

Please read the Operator SDK doc to understand what Operator is.

For your devel enviroment, you could start with [CodeReady](https://github.com/code-ready/crc "CodeReady Containers") enviroment, which you have cluster admin role to deploy CRDs.

For update the ansible role, check and update in `roles/reportportal`.

Then follow the [Ansible user guide](https://github.com/operator-framework/operator-sdk/blob/master/doc/ansible/user-guide.md "Ansible User Guide for Operator SDK") doc with build and test the operator.

CI
--

To be add
