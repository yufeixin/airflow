 .. Licensed to the Apache Software Foundation (ASF) under one
    or more contributor license agreements.  See the NOTICE file
    distributed with this work for additional information
    regarding copyright ownership.  The ASF licenses this file
    to you under the Apache License, Version 2.0 (the
    "License"); you may not use this file except in compliance
    with the License.  You may obtain a copy of the License at

 ..   http://www.apache.org/licenses/LICENSE-2.0

 .. Unless required by applicable law or agreed to in writing,
    software distributed under the License is distributed on an
    "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
    KIND, either express or implied.  See the License for the
    specific language governing permissions and limitations
    under the License.

Production Guide
================

The following are things to consider when using this Helm chart in a production environment.

Database
--------

You will want to use an external database instead of the one deployed with the chart by default.
Both **PostgresSQL** and **MySQL** are supported. Supported versions can be
found on the :doc:`Set up a Database Backend <apache-airflow:howto/set-up-database>` page.

.. code-block:: yaml

  # Don't deploy postgres
  postgre:
    enabled: false

  # Use an external database
  data:
    metadataConnection:
      user: ...
      pass: ...
      protocol: postgresql  # or 'mysql'
      host: ...
      port: ...
      db: ...

PgBouncer
---------

If you are using PostgresSQL as your database, you will likely want to enable `PgBouncer <https://www.pgbouncer.org/>`_ as well.
Airflow can open a lot of database connections due to its distributed nature and using a connection pooler can significantly
reduce the number of open connections on the database.

.. code-block:: yaml

  pgbouncer:
    enabled: true

Depending on the size of you Airflow instance, you may want to adjust the following as well (defaults are shown):

.. code-block:: yaml

  pgbouncer:
    # The maximum number of connections to PgBouncer
    maxClientConn: 100
    # The maximum number of server connections to the metadata database from PgBouncer
    metadataPoolSize: 10
    # The maximum number of server connections to the result backend database from PgBouncer
    resultBackendPoolSize: 5

DAG Files
---------

See :doc:`manage-dags-files`.

Accessing the Airflow UI
------------------------

How you access the Airflow UI will depend on your environment, however the chart does support various options:

Ingress
^^^^^^^

You can create and configure ``Ingress`` objects. See the :ref:`Ingress chart parameters <parameters:ingress>`.
For more information on ``Ingress``, see the
`Kubernetes Ingress documentation <https://kubernetes.io/docs/concepts/services-networking/ingress/>`_.

LoadBalancer Service
^^^^^^^^^^^^^^^^^^^^

You can change the Service type for the webserver to be ``LoadBalancer``, and set any necessary annotations:

.. code-block:: yaml

  webserver:
    service: LoadBalancer
    annotations: {}

For more information on ``LoadBalancer`` Services, see the `Kubernetes LoadBalancer Service Documentation
<https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer>`_.

Logging
-------

Depending on your choice of executor, task logs may not work out of the box. All logging choices can be found
at :doc:`manage-logs`.

Metrics
-------

The chart can support sending metrics to an existing StatsD instance or provide a Prometheus endpoint.

Prometheus
^^^^^^^^^^

The metrics endpoint is available at ``svc/{{ .Release.Name }}-statsd:9102/metrics``.

External StatsD
^^^^^^^^^^^^^^^

To use an external StatsD instance:

.. code-block:: yaml

  statsd:
    enabled: false
  config:
    metrics:  # or 'scheduler' for Airflow 1
      statsd_on: true
      statsd_host: ...
      statsd_port: ...

Celery Backend
--------------

If you are using ``CeleryExecutor`` or ``CeleryKubernetesExecutor``, you can bring your own Celery backend.

By default, the chart will deploy Redis. However, you can use any supported Celery backend instead:

.. code-block:: yaml

  redis:
    enabled: false
  data:
    brokerUrl: redis://redis-user:password@redis-host:6379/0

For more information about setting up a Celery broker, refer to the
exhaustive `Celery documentation on the topic <http://docs.celeryproject.org/en/latest/getting-started/>`_.
