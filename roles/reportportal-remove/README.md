Role Name
=========

Remove Operands deployed by ReportPortal Operator.

Requirements
------------

Install ansible openshift module with dnf or pip
```
$ sudo dnf install python3-openshift

or

$ pip install openshift
```

Install `oc` client in your host with [doc](https://docs.openshift.com/container-platform/4.3/cli_reference/openshift_cli/getting-started-cli.html "Getting started cli")

Make sure you are login to your OpenShift cluster before run the role or export k8s login credential variables in env.

Role Variables
--------------

The OpenShift project namespace:

    namespace: your_openshift_project_name

ReportPortal dependency services values

    remove_rabbitmq: yes
    remove_postgresql: yes
    remove_elasticsearch: yes
    remove_minio: no

By default don't remove Minio as it's used as data backup target.

License
-------

Apache 2.0
