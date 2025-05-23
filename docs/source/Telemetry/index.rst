Telemetry and visualizations
------------------------------

The telemetry feature allows the set up  of Omnia telemetry (to poll values from all Omnia provisioned nodes in the cluster) and/or iDRAC telemetry (To poll values from all eligible iDRACs in the cluster). It also installs `Grafana <https://grafana.com/>`_ and `Loki <https://grafana.com/oss/loki/>`_ as Kubernetes pods.

.. note:: In order to enable telemetry feature in Omnia, ensure to add ``telemetry`` in ``software_config.json``.

To initiate telemetry support, fill out the following parameters in ``input/telemetry_config.yml``:

.. csv-table:: Parameters
   :file: ../Tables/telemetry_config.csv
   :header-rows: 1
   :keepspace:

.. [1] Boolean parameters do not need to be passed with double or single quotes.

.. note:: The ``input/telemetry_config.yml`` file is encrypted during the execution of ``omnia.yml`` playbook. Use the below commands to edit the encrypted input files:
    ::
        ansible-vault edit telemetry_config.yml --vault-password-file .telemetry_vault_key

Once you have executed ``discovery_provision.yml`` and has also provisioned the cluster, initiate telemetry on the cluster as part of ``omnia.yml``, which configures the cluster with scheduler, storage and authentication using the below command. ::

    ansible-playbook omnia.yml -i inventory

Optionally, you can initiate only telemetry using the below command: ::

    ansible-playbook telemetry.yml -i inventory

.. note::

    * To run the ``telemetry.yml`` playbook independently from the ``omnia.yml`` playbook on Intel Gaudi nodes, start by executing the ``performance_profile.yml`` playbook. Once that’s done, you can run the ``telemetry.yml`` playbook separately.

    * Depending on the type of telemetry initiated, include the following possible groups in the inventory:

        * omnia_telemetry: ``slurm_control_node``, ``slurm_node``, ``login``, ``kube_control_plane``, ``kube_node``, ``etcd``, ``auth_server``

        * idrac_telemetry: ``idrac``

        * k8s_telemetry on Prometheus: ``kube_control_plane``, ``kube_node``, ``etcd``

    * If you would like a local backup of the timescaleDB used to store telemetry data, `click here <../Utils/timescaledb_utility.html>`_.

After initiation, new iDRACs can be added for ``idrac_telemetry`` acquisition by running the following commands: ::

    ansible-playbook add_idrac_node.yml -i inventory

**Modifying telemetry information**

To modify how data is collected from the cluster, modify the variables in ``omnia/input/telemetry_config.yml`` and re-run the ``telemetry.yml`` playbook.

* When ``omnia_telemetry_support`` is set to false, Omnia Telemetry Acquisition service will be stopped on all cluster nodes provided in the passed inventory.
* When ``omnia_telemetry_support`` is set to true, Omnia Telemetry Acquisition service will be restarted on all cluster nodes provided in the passed inventory.
* To start or stop the collection of regular metrics, health check metrics, or GPU metrics, update the values of ``collect_regular_metrics``, ``collect_health_check_metrics``, or ``collect_gpu_metrics``. For a list of all metrics collected, `click here <TelemetryMetrics.html>`_.

.. note::
    * Currently, changing the ``grafana_username`` and ``grafana_password`` values is not supported via ``telemetry.yml``.
    * The passed inventory should have an idrac group, if ``idrac_telemetry_support`` is true.
    * If ``omnia_telemetry_support`` is true, then the inventory should have OIM and cluster node groups (as specified in the sample files) along with optional login group.
    * If a subsequent run of ``telemetry.yml`` fails, the ``telemetry_config.yml`` file will be unencrypted.

**To access the Grafana UI**

*Prerequisites*

    * ``visualisation_support`` should be set to true when running ``telemetry.yml`` or ``omnia.yml``.


i. Find the IP address of the Grafana service using ``kubectl get svc -n grafana``


.. image:: ../images/grafanaIP.png


ii. Login to the Grafana UI by connecting to the cluster IP of grafana service obtained above via port 5000, that's ``http://xx.xx.xx.xx:5000/login``


.. image:: ../images/Grafana_login.png


iii. Enter the ``grafana_username`` and ``grafana_password`` as mentioned in ``input/telemetry_config.yml``.


.. image:: ../images/Grafana_Dashboards.png


Loki log collections can viewed on the explore section of the grafana UI.


.. image:: ../images/Grafana_Loki.png


Datasources configured by Omnia can be viewed as seen below.


.. image:: ../images/GrafanaDatasources.png

**To use Loki for log filtering**

    i. Login to the Grafana UI by connecting to the cluster IP of grafana service obtained above via port 5000. That is ``http://xx.xx.xx.xx:5000/login``

    ii. In the Explore page, select **control-plane-loki**.

    .. image:: ../images/Grafana_ControlPlaneLoki.png

    iii. The log browser allows you to filter logs by job, node and/or user.

Example ::

    (job)= "cluster deployment logs") |= "nodename"
    (job="compute log messages") |= "nodename" |="node_username"


**To use Grafana to view telemetry data**

    i. Login to the Grafana UI by connecting to the cluster IP of grafana service obtained above via port 5000. That is ``http://xx.xx.xx.xx:5000/login``

    ii. In the Explore page, select **telemetry-postgres**.

    .. image:: ../images/Grafana_Telemetry_PostGRES.png

    iii. The query builder allows you to create SQL commands that can be used to query the ``omnia_telemetry.metrics`` table. Filter the data required using the following fields:

        * **id**: The name of the metric.
        * **context**: The type of metric being collected (Regular Metric, Health Check Metric and GPU metric).
        * **label**: A combined field listing the **id** and **context** row values.
        * **value**: The value of the metric at the given timestamp.
        * **unit**: The unit measure of the metric (eg: Seconds, kb, percent, etc.)
        * **system**: The service tag of the cluster node.
        * **hostname**: The hostname of the cluster node.
        * **time**: The timestamp at which the metric was polled from the cluster node.

    *iDRAC telemetry data from Grafana*

    .. image:: ../images/idractelemetry.png

.. note:: If you are more comfortable using SQL queries over the query builder, click on **Edit SQL** to directly provide your query. Optionally, the data returned from a query can be viewed as a graph.

**Visualizations**

If ``idrac_telemetry_support`` and ``visualisation_support`` is set to true, Parallel Coordinate graphs can be used to view system statistics.

.. toctree::
    Visualizations/index
    TelemetryMetrics
    MetricInfo
    TimescaleDB
    Prometheus_k8s
    Gaudi_metrics




