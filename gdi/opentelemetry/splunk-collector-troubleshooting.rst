.. _otel-splunk-collector-tshoot:

****************************************************************
Troubleshoot the Splunk OpenTelemetry Collector
****************************************************************

.. meta::
      :description: Describes known issues when using the Splunk Distribution of OpenTelemetry Collector.

See the following issues and workarounds for the Splunk Distribution of OpenTelemetry Collector.

Collector isn't behaving as expected
=========================================

The Collector might experience the issues described in this section.

Collector or td-agent service isn't working
--------------------------------------------------

If either the Collector or td-agent services are not installed and configured, check these things to fix the issue:

* Check that the OS is supported.
* Check that systemd is installed (if using Linux).
* Check that your platform is not running in a containerized environment.
* Check the installation logs for more details.

Collector exits or restarts
-----------------------------------------

The collector might exit or restart for the following reasons:

* Memory pressure due to a missing or misconfigured ``memory_limiter`` processor
* Improperly sized for load
* Improperly configured. For example, a queue size configured higher than available memory.
* Infrastructure resource limits. For example, Kubernetes.

Restart the Splunk Distribution of OpenTelemetry Collector and check the configuration.

Collector doesn't start in Windows Docker containers
-----------------------------------------------------------

The process might fail to start in a custom built, Windows-based Docker container, resulting in a "The service process could not connect to the service controller" error message.

In this case, the ``NO_WINDOWS_SERVICE=1`` environment variable must be set to force the Splunk Distribution of OpenTelemetry Collector to start as if it were running in an interactive terminal, without attempting to run as a Windows service.

Collector is experiencing data issues
============================================

The Collector might experience the issues described in this section.

Collector is dropping data
--------------------------------

Data might drop for a variety of reasons, but most commonly for the following reasons:

* The collector is improperly sized, resulting in the Splunk Distribution of OpenTelemetry Collector being unable to process and export the data as fast as it is received. See :ref:`otel-sizing` for sizing guidelines.
* The exporter destination is unavailable or accepting the data too slowly. To mitigate drops, configure the ``batch`` processor. In addition, you might also need to configure the queued retry options on activated exporters.

Collector isn't receiving data
-------------------------------------

The collector might not receive data for the following reasons:

* Network configuration issues
* Receiver configuration issues
* The receiver is defined in the receivers section, but not activated in any pipelines.
* The client configuration is incorrect

Check the logs and :new-page:`Troubleshooting zPages <https://github.com/open-telemetry/opentelemetry-collector/blob/main/docs/troubleshooting.md#zpages>` in GitHub for more information.

Collector can't process data
-----------------------------------

The collector might not process data for the following reasons:

* The attributes processors work only for "tags" on spans. The span name is handled by the span processor.
* Processors for trace data (except tail sampling) only work on individual spans. Make sure your collector is configured properly.

Collector can't export data
------------------------------------

The collector might be unable to export data for the following reasons:

* Network configuration issues, such as firewall, DNS, or proxy support.
* Incorrect exporter configuration
* Incorrect credentials
* The destination is unavailable

If you need to use a proxy, see :ref:`configure-proxy-collector`.

Check the logs and :new-page:`Troubleshooting zPages <https://github.com/open-telemetry/opentelemetry-collector/blob/main/docs/troubleshooting.md#zpages>` in GitHub for more information.

.. _collector-gateway-metrics-issue:

Metrics and metadata not available in Gateway mode
============================================================

If you don't see metrics and metadata after manually deploying the Collector in Gateway mode, the agent configuration might be lacking pipelines that use the SignalFx exporter. Follow these steps to review the configuration:

#. Make sure that your Gateway can listen to requests on ports 6060 and 9943.

#. Make sure that the agent configuration has a ``signalfx`` exporter in a pipeline. The following example shows a ``signalfx`` exporter and a pipeline that uses it for sending metrics:

   .. code-block:: yaml
      :emphasize-lines: 2,3,4,5,14

      exporters:
         signalfx:
            access_token: "${SPLUNK_ACCESS_TOKEN}"
            api_url: "http://${SPLUNK_GATEWAY_URL}:6060"
            ingest_url: "http://${SPLUNK_GATEWAY_URL}:9943"
            sync_host_metadata: true
            correlation:
         # Other exporters

      service:
         extensions: [health_check, http_forwarder, zpages]
         pipelines:
            metrics/internal:
                  receivers: [prometheus/internal]
                  processors: [memory_limiter, batch, resourcedetection]
                  exporters: [signalfx]
            # Other pipelines

Report host metrics in APM
==================================

To capture and send relevant data to show correlated infrastructure metrics in the APM service, add the ``resource/add_environment`` processor to your configuration.

This processor inserts a ``deployment.environment`` span tag to all spans. The APM charts require the environment span tag to be set correctly. Configure this span tag in the instrumentation, but if that is not an option, you can use this processor to insert the required ``deployment.environment`` span tag value.

For example:

.. code-block:: yaml

    processors:
      resourcedetection:
        detectors: [system,env,gce,ec2]
        override: true
      resource/add_environment:
        attributes:
          - action: insert
            value: staging
            key: deployment.environment

Extract a running configuration
=========================================
Extracting a running configuration saves or stores the contents of a configuration file to logs that you can use to troubleshoot issues. You can extract a running configuration by accessing these ports:

* ``http://localhost:55554/debug/configz/initial``
* ``http://localhost:55554/debug/configz/effective``

For Linux, the support bundle script captures this information. See :ref:`otel-install-linux` for the installer script. This capability is primarily useful if you are using remote configuration options such as Zookeeper where the startup configuration can change during operation.

Check metric data from the command line
==============================================

To check whether host metrics are being collected and processed correctly, you can query the Collector for raw data using ``curl`` or similar tools from the command line.

- On Linux, run ``curl http://localhost:8888/metrics`` in your terminal.
- On Windows, run ``"Invoke-WebRequest -URI https://localhost:8888/metrics"`` in PowerShell.

You can then pipe the output to ``grep`` (Linux) or ``Select-String`` (Windows) to filter the data. For example, ``curl http://localhost:8888/metrics | grep service_instance_id`` retrieves the service instance ID.

You're getting a "bind: address already in use" error message
==================================================================================

If you see an error message such as "bind: address already in use", another resource is already using the port that the current configuration requires. This resource could be another application, or a tracing tool such as Jaeger or Zipkin.

You can modify the configuration to use another port. You can modify any of these endpoints or ports:

* Receiver endpoint
* Extensions endpoint
* Metrics address (if port 8888)

If you see this error message on Kubernetes and you're using Helm charts, modify the configuration by updating the chart values for both configuration and exposed ports.

You're getting a "pattern not matched" error message
==================================================================================

If you see an error message such as "pattern not matched", this message is from Fluentd, and means that the ``<parser>`` was unable to match based on the log message. As a result, the log message is not collected. Check the Fluentd configuration and update as required.

You're receiving an HTTP error code
==================================================================================

If an HTTP request is not successfully completed, you might see the following HTTP error codes.

.. list-table::
   :widths: 50 50
   :header-rows: 1

   * - Error code
     - Description
   * - ``401 (UNAUTHORIZED)``
     - Configured access token or realm is incorrect.
   * - ``404 (NOT FOUND)``
     - Incorrect configuration parameter, like an endpoint or path, or a network, firewall, or port issue.
   * - ``429 (TOO MANY REQUESTS)``
     - Organization is not provisioned for the amount of traffic being sent. Reduce traffic or request increase in capacity.
   * - ``503 (SERVICE UNAVAILABLE)``
     - If using the Log Observer, this is the same as the ``429 (TOO MANY REQUESTS)`` error code, due to how HECv1 responds. Otherwise, check the status page.

Log collection issues
=========================================

See the OpenTelemetry project documentation :new-page:`to activate the Collector's debug logging <https://github.com/open-telemetry/opentelemetry-collector/blob/main/docs/troubleshooting.md#logs>`.

Here are some common issues related to log collection on the Collector.

Source isn't generating logs
----------------------------------------

If using Linux, run the following commands to check if the source is generating logs:

.. code-block:: bash

  tail -f /var/log/myTestLog.log
  journalctl -u my-service.service -f


If using Windows, run the following command to check if the source is generating logs:

.. code-block:: shell

  Get-Content myTestLog.log 

Fluentd isn't configured correctly
----------------------------------------

Do the following to check the Fluentd configuration:

#. Check that td-agent is running. On Linux, run ``systemctl status td-agent``. On Windows, run ``Get-Service td-agent``.
#. If you changed the configuration, restart Fluentd. On Linux, run ``systemctl restart td-agent``. On Windows, run ``Restart-Service -Name td-agent``.
#. Check fluentd.conf and conf.d/\*. ``@label @SPLUNK`` must be added to every source to activate log collection.
#. Manual configuration might be required to collect logs off the source. Add configuration files to in the conf.d directory as needed.
#. Activate debug logging in fluentd.conf (``log_level debug``), restart td-agent, and check that the source is generating logs.

While every attempt is made to properly configure permissions, it is possible that td-agent does not have the permission required to collect logs. Debug logging should indicate this issue.

It is possible that the ``<parser>`` section configuration does not match the log events.

If you see a message such as "2021-03-17 02:14:44 +0000 [debug]: #0 connect new socket", Fluentd is working as expected. You need to activate debug logging to see this message.

Collector isn't configured properly
----------------------------------------

Do the following to check the Collector configuration:

#. Go to ``http://localhost:55679/debug/tracez`` to check zPages for samples. You might need to configure the endpoint.
#. Activate logging exporter.
#. Run ``journalctl -u splunk-otel-collector.service -f`` to collect the logs for you to review.
#. Review :ref:`otel-splunk-collector-tshoot` if you can't find what you need in the logs.

Test the Collector by sending synthetic data
------------------------------------------------------------
You can manually generate logs. By default, Fluentd monitors journald and /var/log/syslog.log for events.

.. code-block:: bash

   echo "2021-03-17 02:14:44 +0000 [debug]: test" >>/var/log/syslog.log
   echo "2021-03-17 02:14:44 +0000 [debug]: test" | systemd-cat

.. note::

   Properly structured syslog is required for Fluentd to properly pick up the log line.

.. _unwanted_profiling_logs:

Unwanted profiling logs appearing in Observability Cloud
------------------------------------------------------------

By default, the Splunk Distribution of the OpenTelemetry Collector sends AlwaysOn Profiling data through a ``logs`` pipeline that uses the Splunk HEC exporter.

To send logs to Splunk Observability Cloud without AlwaysOn Profiling data, see :ref:`no_profiling_data`.

.. _disable_log_collection:

Exclude log data in the Collector
------------------------------------------------------------

By default, the Splunk Distribution of the OpenTelemetry Collector collects and send logs to Observability Cloud through a logs pipeline that uses the Splunk HEC exporter. See :ref:`splunk-hec-exporter` for more information.

To avoid sending log data through the Collector to Splunk Observability Cloud, see :ref:`exclude-log-data`.

Send logs from the Collector to Splunk Cloud Platform or Enterprise
-----------------------------------------------------------------------

To send logs from the Collector to Splunk Cloud Platform or Splunk Enterprise, see :ref:`send_logs_to_splunk`.

Trace collection issues
================================

Here are some common issues related to trace collection on the Collector.

Test the Collector by sending synthetic data
------------------------------------------------------------

You can test the Collector to make sure it can receive spans without instrumenting an application. By default, the Collector activates the Zipkin receiver, which is capable of receiving trace data over JSON.

To test the UI, you can submit a POST request or paste JSON in this directory, as shown in the following example.

.. code-block:: bash

   curl -OL https://raw.githubusercontent.com/openzipkin/zipkin/master/zipkin-lens/testdata/yelp.json
   curl -X POST localhost:9411/api/v2/spans -H'Content-Type: application/json' -d @yelp.json

.. note::

   Update the ``localhost`` field as appropriate to reach the Collector.

No response means the request was sent successfully. You can also pass ``-v`` to the curl command to confirm.
