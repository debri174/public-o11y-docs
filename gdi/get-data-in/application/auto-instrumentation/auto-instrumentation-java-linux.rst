.. meta::
   :description: How to activate zero configuration automatic instrumentation for Linux Java applications and thus collect and send traces to Splunk Application Performance Monitoring (APM) without altering your code.

.. include:: /_includes/gdi/zero-config-preview-header.rst


.. _auto-instrumentation-java-linux:

=======================================================================================
Zero Configuration Auto Instrumentation for Linux Java applications
=======================================================================================

Zero Configuration Auto Instrumentation for Java activates automatic instrumentation for Linux Java applications. To activate automatic instrumentation, install the ``splunk-otel-auto-instrumentation`` package. After installing the package, you must start or restart any Java applications that you want to instrument.

.. _prerequisites:

Prerequisites
===========================

.. include:: /_includes/gdi/zero-conf-reqs.rst

.. _install-the-package:

Install the package
=========================================================

You can install the ``splunk-otel-auto-instrumentation`` package in two ways:

- Install using the Collector installer script
- Install using Debian or RPM packages

.. tabs::

   .. tab:: Install using the Collector installer script

      To install the package, run the Collector installer script with the ``--with-instrumentation`` option. The installer script will install the Collector and the Java agent from the Splunk Distribution of OpenTelemetry Java. The Java agent is then loaded automatically when a Java application starts on the local machine.

      Run the installer script with the ``--with-instrumentation`` option, as shown in the following example. Replace  ``<SPLUNK_REALM>`` and ``<SPLUNK_ACCESS_TOKEN>`` with your Observability Cloud realm and token, respectively.

      .. code-block:: bash

         curl -sSL https://dl.signalfx.com/splunk-otel-collector.sh > /tmp/splunk-otel-collector.sh && \
         sudo sh /tmp/splunk-otel-collector.sh --with-instrumentation --realm <SPLUNK_REALM> -- <SPLUNK_ACCESS_TOKEN>

      To automatically define the optional ``deployment.environment`` resource attribute at installation time, run the installer script with the ``--deployment-environment <VALUE>`` option. Replace ``<VALUE>`` with the desired attribute value, for example, ``prod``, as shown in the following example:

      .. code-block:: bash

         curl -sSL https://dl.signalfx.com/splunk-otel-collector.sh > /tmp/splunk-otel-collector.sh && \
         sudo sh /tmp/splunk-otel-collector.sh --with-instrumentation --deployment-environment prod --realm <SPLUNK_REALM> -- <SPLUNK_ACCESS_TOKEN>

      
      Next, :ref:`ensure the service is running<verify-install>` and :ref:`start your applications<start-restart-java-apps>`. 


   .. tab::  Install using Debian or RPM packages

      .. note:: You must first install the Splunk OpenTelemetry Collector using the :ref:`linux-packages`.

      After installing the Collector, follow these steps to install the package using the Debian or RPM repositories with ``root`` privileges:

      1. You can either download the ``splunk-otel-auto-instrumentation`` package directly from the :new-page:`GitHub Releases page <https://github.com/signalfx/splunk-otel-collector/releases>` or add the Splunk repository to the package repositories on your Linux host. See :new-page:`Debian or RPM package repositories <https://docs.splunk.com/Observability/gdi/opentelemetry/install-linux.html#debian-or-rpm-packages>` for instructions on how to configure your package repository.

      2. Run the following commands to install the package. Replace ``<path to splunk-otel-auto-instrumentation deb/rpm>`` with the local path to the downloaded package.

         .. tabs::

            .. code-tab:: bash Debian
               
               sudo dpkg -i <path to splunk-otel-auto-instrumentation deb>
               
            .. code-tab:: bash RPM
               
               sudo rpm -ivh <path to splunk-otel-auto-instrumentation rpm>

      3. Edit the ``/etc/otel/collector/splunk-otel-collector.conf`` file to set the ``SPLUNK_ACCESS_TOKEN`` and ``SPLUNK_REALM`` variables to the values you got earlier. If the file does not exist, use the provided sample at ``/etc/otel/collector/splunk-otel-collector.conf.example`` as a starting point.

         .. code-block:: bash

            SPLUNK_ACCESS_TOKEN=<access_token>
            SPLUNK_REALM=<realm>

      4. Start the collector service:

         .. code-block:: bash

            sudo systemctl start splunk-otel-collector


      5. :ref:`verify-install`.
      6. :ref:`start-restart-java-apps`.


.. _verify-install:

Ensure the service is running
--------------------------------------------------------

After a successful installation, run the following command to ensure the ``splunk-otel-collector`` service is running:

.. code-block:: bash

   sudo systemctl status splunk-otel-collector

If the service is not running, start or restart it with the following command:

.. code-block:: bash

   sudo systemctl restart splunk-otel-collector

If the service fails to start, check that the ``SPLUNK_REALM`` and ``SPLUNK_ACCESS_TOKEN`` in ``/etc/otel/collector/splunk-otel-collector.conf`` are correct. You can also view the service logs with this command:

.. code-block:: bash

   sudo journalctl -u splunk-otel-collector


.. _start-restart-java-apps:

Start your applications
-----------------------------------

You are now ready to start sending traces to Splunk APM. For auto instrumentation to take effect, you must manually start or restart any Java applications on the host where you installed the package. This is true after installing the auto instrumentation package for the first time and whenever you make any changes to the configuration file. 

Once your applications are running, you can :ref:`verify-apm-data`. You can also :ref:`optionally configure instrumentation settings<configure-java-zeroconf-linux>`. 

.. _configure-java-zeroconf-linux:

Optionally configure instrumentation
====================================

The default settings for zero config autoinstrumentation are sufficient for most basic cases. You can set additional customizations by editing the configuration file. 

Installing the package installs the following artifacts:

- The configuration file at ``/usr/lib/splunk-instrumentation/instrumentation.conf`` 
- The :new-page:`Java Instrumentation Agent <https://github.com/signalfx/splunk-otel-java>` at ``/usr/lib/splunk-instrumentation/splunk-otel-javaagent.jar``
- The shared instrumentation library at ``/usr/lib/splunk-instrumentation/libsplunk.so```

The following is a sample of the default configuration file:

.. code-block:: bash

   java_agent_jar=/usr/lib/splunk-instrumentation/splunk-otel-javaagent.jar

By default, the configuration file only specifies one parameter, ``java_agent_jar``, which points to the path of the installed Java Instrumentation Agent.

The :ref:`supported-parameters` section discusses additional parameters you can set in the configuration file.

.. _supported-parameters:

Supported parameters
---------------------------------------------

The following table shows the supported parameters for the ``/usr/lib/splunk-instrumentation/instrumentation.conf`` file.

.. list-table::       
   :header-rows: 1
   :widths: 20 60 20
   :width: 100%
   
   * - Parameter
     - Description
     - Required
   * - ``java_agent_jar``
     - The full path to the JAR file provided by the installer.
     -  Yes
   * - ``service_name``
     - An optional parameter that specifies a unique identifier for a particular host. If you set this parameter, all instrumented Java applications on the host have their service name set using the  ``OTEL_SERVICE_NAME`` environment variable. If this parameter is not set, the shared object assigns a generated name. 
     - No
   * - ``resource_attributes`` 
     - Contains a comma-separated list of name-value pairs of the form ``name=value``. Use this attribute to add extra tags to the generated trace data. For example, you might want every span to include a ``build.id`` set to ``feb2023`` and a ``deployment.environment`` set to ``test``. You can set the ``resource_attributes`` to ``build.id=feb2023_v2,deployment.environment=test``. The shared object sets the ``RESOURCE_ATTRIBUTES`` environment variable to the value you specified, and the Java agent includes each pair in every span.
     - No  


Keep the following information in mind after the installation:

-  Whenever you change the configuration file, you must manually start or restart any Java applications on the host where you installed the package.

- If you installed the package with the installer script and specified the ``--deployment-environment VALUE`` when you ran the script, the ``deployment.environment=VALUE`` resource attribute is automatically added to the configuration file.

- The ``/etc/ld.so.preload`` file is automatically created or updated with the default path to the installed instrumentation library. If necessary, custom library paths can be manually added to this file.

**Advanced configuration**

More advanced configuration options like correlating traces and logs and enabling custom sampling are available by :ref:`configuring the Java agent<advanced-java-otel-configuration>`. 


.. _upgrade-the-package:

Upgrade the package
==========================

You can upgrade the package in two ways:

- :ref:`upgrade-with-script`
- :ref:`upgrade-manually`

.. _upgrade-with-script:

Upgrade using the package repository
---------------------------------------------

If you installed the package using the installer script, or if you configured the Debian or RPM package repositories manually, run the following commands according to your platform. Upgrading the package requires ``root`` privileges. 

Debian
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Run the following commands:

.. code-block:: bash

   sudo apt-get update
   sudo apt-get --only-upgrade splunk-otel-auto-instrumentation

You might see a prompt to keep or overwrite the configuration file at ``/usr/lib/splunk-instrumentation/instrumentation.conf``. If you choose to overwrite, the configuration file reverts to the default file provided by the upgraded package.

RPM
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For the RPM package management system, run the following commands:

yum
##############################

.. code-block:: bash

   sudo yum upgrade splunk-otel-auto-instrumentation

dnf
###################

.. code-block:: bash

   sudo dnf upgrade splunk-otel-auto-instrumentation

zypper
################

.. code-block:: bash

   sudo zypper refresh
   sudo zypper update splunk-otel-auto-instrumentation

After you've upgraded the packages, manually start or restart the Java applications on the host for the changes to take effect.

.. _upgrade-manually:

Upgrade using Debian or RPM packages
---------------------------------------------

To manually upgrade the package:

1. Download the ``splunk-auto-auto-instrumentation`` Debian or RPM package for the target system from the :new-page:`GitHub Releases page <https://github.com/signalfx/splunk-otel-collector/releases>`.

2. Run the following commands to install the package. Replace ``<path to splunk-otel-auto-instrumentation deb/rpm>`` with the local path to the downloaded package:

   .. tabs::

      .. code-tab:: bash Debian
      
         sudo dpkg -i <path to splunk-otel-auto-instrumentation deb>
      
      .. code-tab:: bash RPM
      
         sudo rpm -Uvh <path to splunk-otel-auto-instrumentation rpm>

After upgrading the Debian package, you might see a prompt to keep or overwrite the configuration file at ``/usr/lib/splunk-instrumentation/instrumentation.conf``. If you choose to overwrite, the configuration file reverts to the default file provided by the upgraded package.

You can also upgrade using the same package repositories as the Collector. See :new-page:`Debian or RPM packages <https://docs.splunk.com/Observability/gdi/opentelemetry/install-linux.html#debian-or-rpm-packages>` for more information.

Deactivate automatic instrumentation
=========================================

Use one of the following options to deactivate automatic instrumentation:

- Uninstall the package by running the following command:

  .. code-block:: bash
   
     curl -sSL https://dl.signalfx.com/splunk-otel-collector.sh > /tmp/splunk-otel-collector.sh && \
     sudo sh /tmp/splunk-otel-collector.sh --uninstall
  
  See :ref:`otel-linux-uninstall-otel-and-tdagent` for more information on the files deleted by the uninstall.

- Set ``DISABLE_SPLUNK_AUTOINSTRUMENTATION`` to any nonempty value other than ``false``, ``FALSE``, or ``0``.

- Set the ``JAVA_TOOL_OPTIONS`` environment variable to some value that you want the JVM to pick up.

- Delete or move the ``instrumentation.conf`` configuration file.


.. include:: /_includes/gdi/next-steps.rst


Troubleshooting
=========================================

If you activate auto instrumentation and you see an error message or you do not see any data in Observabiity Cloud APM, try the following steps:

- Check the  ``splunk-otel-collector`` service logs:
  
.. code-block:: bash

   sudo journalctl -u splunk-otel-collector

- You can also follow the :ref:`steps to activate trace and troubleshoot the Java agent<basic-java-troubleshooting>`.


.. include:: /_includes/troubleshooting-steps.rst