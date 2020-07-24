Ansible Role: ReportPortal Openshift
====================================

Deploy ReportPortal v5 on OpenShift Container Platform.

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

ReportPortal gateway domain values:

    app_domain: apps.ocp.test.example.com

ReportPortal dependency services values

RabbitMQ deploy parameters:

    deploy_rabbitmq: yes
    rabbitmq_image: docker.io/bitnami/rabbitmq:3.8.3-debian-10-r2
    rabbitmq_volume_size: 8Gi
    load_definition: "{{ lookup('template', 'load-definition.json') | to_json | string | b64encode }}"

If deploy_rabbimq is `yes`, RabbitMQ service will also be deployed in current project namespace and don't need update the following default values.

RabbitMQ service default values:

    rabbitmq_name: rabbitmq
    rabbitmq_host: rabbitmq
    rabbitmq_port: 5672
    rabbitmq_user: rabbitmq
    rabbitmq_password: rp_rabbitmq
    rabbitmq_apiuser: rabbitmq
    rabbitmq_apiport: 15672
    rabbitmq_secret: rabbitmq

If deploy_rabbitmq is `no`, the values need be updated to use existing rabbitmq service.

PostgreSQL deploy parameters:

    deploy_postgresql: yes
    db_storage_size: 8Gi
    pg_image: docker.io/bitnami/postgresql:11.7.0-debian-10-r9

If deploy_postgresql is `yes`, PostgreSQL service will also be deployed in current project namespace and don't need update the following default values.

PostgreSQL service default values:
rp_db: reportportal
rp_db_user: postgres
rp_db_password: lYHfhS2Hfy
rp_db_host: postgresql
rp_db_port: 5432
rp_db_secret: postgresql

If deploy_postgresql is `no`, the values need be updated to use existing PostgreSQL service.

Minio deploy parameters:

    deploy_minio: yes
    minio_image: minio/minio:RELEASE.2019-08-07T01-59-21Z
    minio_storage_size: 10Gi
    cloud_replicas: 4

If deploy_minio is `yes`, Minio service will also be deployed in current project namespace and don't need update the following default values.

Mino service default values:

    minio_endpoint: http://minio:9000
    accesskey: AKIAIOSFODNN7EXAMPLE
    secretkey: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
    config_path: "/data/.minio"
    config_pathmc: "/data/.mc"
    mount_path: "/export"

    s3_enabled: false
    s3_endpoint: ""

    azure_enabled: false

    gcs_enabled: false
    gcs_key_json: ""
    gcs_projectid: ""

    oss_enabled: false
    oss_endpoint: ""

If deploy_minio is `no`, the values need be updated to use existing Minio service.

Elasticsearch deploy parameters:

    deploy_elasticsearch: yes
    es_replicas: 1          # need update to 3 for ha
    es_storage_size: 10Gi   # better 2update to 30
    es_image: docker.elastic.co/elasticsearch/elasticsearch:7.6.1

If deploy_elasticsearch is `yes`, Elasticsearch service will also be deployed in current project namespace and don't need update the following default values.

Elasticsearch service default values:

    es_host: elasticsearch-master
    es_port: 9200

If deploy_elasticsearch is `no`, the values need be updated to use existing Elasticsearch service.

ReportPortal services values

index default values:

    index_replicas: 1
    index_image: reportportal/service-index:5.0.7

uat default values:

    uat_replicas: 1
    uat_image: docker-registry.upshift.redhat.com/ccit/rp-service-authorization:5.1.0

ui default values:

    ui_replicas: 1
    ui_image: reportportal/service-ui:5.1.0

api default values:

    api_replicas: 1
    api_image: quay.io/waynesun09/service-api:5.1.0-rootless
    rp_amqp_queues: 10
    rp_amqp_queuesperpod: 10

migration default values:

    migration_image: quay.io/waynesun09/rp-migration:5.1.0-bcrypt

analyzer default values:

    analyzer_replicas: 1
    analyzer_image: reportportal/service-auto-analyzer:5.1.0

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      vars:
        - namespace: your_openshift_project_name
        - app_domain: your_openshift_app_domain_name
      roles:
         - role: waynesun09.reportportal_openshift

License
-------

Apache 2.0
