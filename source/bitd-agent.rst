.. Copyright 2018 by Andrei Radulescu-Banu.

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at
 
     http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.

****************
The bitd-agent
****************

.. index::
   single: bitd-agent

.. contents:: Table of Contents

An example run of the bitd-agent
==================================

The main Bitribble application is the ``bitd-agent``. The bitd-agent ``yaml`` or ``xml`` configuration file lists the task instances that the bitd-agent will run. For example:

.. code-block:: none

   bitd-agent -c echo-config.yml

The configuration file describes the task instance parameters, the modules that contain the task implementation, as well as the task instance schedule. Here is the echo-config.yml:

.. code-block:: yaml
   :linenos:

   modules:
     module-name: bitd-echo
   task-inst:
     task-name: echo
     task-inst-name: Echo task inst 1
     schedule:
       type: periodic
       interval: 1s
     args:
       name-int64: -128
       name-int64: 127

   
By default, ``bitd-agent`` will print the task instace results to ``stdout`` in yaml format:

.. code-block:: yaml
   :linenos:

   $ bitd-agent -c modules/echo-config.yml 
   ---
   tags:
     task: echo
     task-instance: Echo task inst 1
   run-id: 0
   run-timestamp: 1539370698248977666
   exit-code: 0
   output:
     name-int64: -128
     name-int64: 127
   ---
   tags:
     task: echo
     task-instance: Echo task inst 1
   run-id: 1
   run-timestamp: 1539370699250138126
   exit-code: 0
   output:
     name-int64: -128
     name-int64: 127
   ---
   ^C...

The task instance will run every second until the ``bitd-agent`` is interrupted with ``Ctrl-C``.

The ``echo`` task just echoes back the input as output. Other task modules are available:

- The ``exec`` task will execute any shell command

- The ``sink-graphite`` task in the ``bitd-sink-graphite`` module will send results directed to it to a Graphite back-end database

- The ``sink-influxdb`` task in the ``bitd-sink-influxdb`` module will send results directed to it to an Influxdb back-end database

Annotated configuration and output
----------------------------------
This section can be skipped on first reading. Here is an annotate version of the configuration file:

.. code-block:: yaml
   :linenos:

   modules: # The list of modules
     module-name: bitd-echo # The module implementing the echo task
   task-inst: # The task instance block
     task-name: echo # The echo task
     task-inst-name: Echo task inst 1 # Multiple task instances can be defined for the same task
     schedule:
       type: periodic # Running periodically
       interval: 1s # ... every second
     args: # The equivalent of Linux process command line arguments
       name-int64: -128
       name-int64: 127

And here is an annotated version of the output:   

.. code-block:: yaml
   :linenos:

   $ bitd-agent -c modules/echo-config.yml 
   ---
   tags: # The task and task instance name are reported back as tags
     task: echo
     task-instance: Echo task inst 1
   run-id: 0 # The run-id is incremented with each task instance run
   run-timestamp: 1539370698248977666 # The Unix time, in nanosecs, when results were reported
   exit-code: 0 # The exit code of the task instance
   output: # The task instance output
     name-int64: -128
     name-int64: 127
   ---
   ^C...

The bitd-agent command line parameters
======================================
The full list of bitd-agent command line parameters are displayed by running

.. code-block:: none

   bitd-agent -h

Configuration file
------------------

The config file is passed in using a ``-c`` parameter. The type of config file can be yaml or xml. The filename suffix is used to detect the syntax used in the file: .yml or .yaml files will be parsed as yaml files, and .xml files will be parsed as xml files.

.. code-block:: none

   -c config_file.{xml|yml|yaml}
     The bitd-agent configuration file. Format is determined from
     file suffix. Default format is yaml.

If you don't wish to rely on the filename suffix to determine the syntax of the file, pass the configuration file used ``-cx`` for xml, and ``-cy`` for yaml:

.. code-block:: none

  -cx config_file.xml
    The bitd-agent configuration file, in xml format.
  -cy config_file.yaml
    The bitd-agent configuration file, in yaml format.

Results display
---------------
The task instance results can be displayed in yaml or xml format. These parameters control the display format, and where the results are saved:

.. code-block:: none

  -r result_file
    The bitd-agent result file. Default: stdout.
  -rx
    Format the result output as xml.
  -ry
    Format the result output as yaml (default).

To discard the results, pass ``-r /dev/null``. If the result file is ``stdout``, results will be sent to ``stdout``. On Unix type systems, including Cygwin, the same effect can be achieved passing ``/dev/stdout`` as a result file - but Windows does not have a notion of ``dev/stdout``. On Windows, it is therefore handy to be about to specify ``stdout`` directly as a parameter to ``-r``.

.. _load-path-parameter:

Module load path
----------------
The modules containing task implementations are DLLs (i.e., Linux shared libraries and Windows dynamically loaded libaries). By default, the ``bitd-agent`` will look for modules in the path specified by the

- ``LD_LIBRARY_PATH`` environment variable on Linux

- ``PATH`` environment variable in Windows Win32 and Cygwin

- ``DYLD_FALLBACK_LIBRARY_PATH`` environment variable on OSX

Additional path lookup folders can be specified with the following parameter:

.. code-block:: none

  -lp load_path
    DLL load library path.

Worker thread pool size
-----------------------
Task instances are executed in the context of threads in a worker thread pool.  If insufficient threads are available, task instances are delayed until a thread becomes available. The default number of threads in the pool rarely needs to be adjusted, but it can be with the following parameter:

.. code-block:: none

  --n-worker-threads thread_count
    Set the max number of worker theads.

Log level
---------
The log level can be controlled using the following options:

.. code-block:: none

  -l|--log-level none|crit|error|warn|info|debug|trace
    Set the log level (default: none).
  -lk|--log-key-level key_name none|crit|error|warn|info|debug|trace
    Set log key level.
  -lf|--log-file file_name
    Save log messages to a file. Default: stdout.
  -ls|--log-file-size size
    Set the log file size. Default: 16777216.
  -lc|--log-file-count count
    Set the log file count. Default: 3.

The ``-l level`` option controls the global log level. Sometimes, when many task instances are running, increasing the global log level results in too many messages. In that case, it is better to keep the global log level set to ``none`` and instead enable the log for specific keys, using ``-lk key_name level``. Each task instance will send its log messages to a specific key. The task instance log key name will be specific to the task instance.

Log levels can also be controlled using the ``config-log`` task located in the ``bitd-config-log`` module. Here is an example configuration enabling logs only for the ``module-mgr`` key (in effect, enabling log messages only from the module-mgr subsystem):

.. code-block:: yaml
   :linenos:

   modules:
     module-name: bitd-config-log
   task-inst:
     task-name: config-log
     task-inst-name: config-log
     schedule:
       type: config
     input:
       log-level: none
       log-key:
         key-name: module-mgr
         log-level: info

This configuration file can be merged into any other configuration file - for example, here it is merged into the echo task instance configuration file:

.. code-block:: yaml
   :linenos:
   :emphasize-lines: 3,13-22

   modules:
     module-name: bitd-echo
     module-name: bitd-config-log
   task-inst:
     task-name: echo
     task-inst-name: Echo task inst 1
     schedule:
       type: periodic
       interval: 1s
     args:
       name-int64: -128
       name-int64: 127
   task-inst:
     task-name: config-log
     task-inst-name: config-log
     schedule:
       type: config
     input:
       log-level: none
       log-key:
         key-name: module-mgr
         log-level: info

The bitd-agent configuration file
=================================
The ``bitd-agent`` configuration can be specified either in ``yaml`` or ``xml`` format. In this section, we will describe the ``yaml`` version of the configuration file. All the examples in this section can be converted to ``xml`` using the ``test-object`` unit tester tool. (Run ``test-object -h`` for options.)

It is easiest to work through the structure of the configuration file through examples. Here is the basic example we will start with:

.. code-block:: yaml
   :linenos:

   modules:
     module-name: bitd-echo
     module-name: bitd-sink-graphite
     module-name: bitd-config-log
   task-inst:
     task-name: config-log
     task-inst-name: config-log
     schedule:
       type: config
     input:
       log-level: warn
       log-key:
         key-name: bit-sink-graphite
         log-level: debug
   task-inst:
     task-name: echo
     task-inst-name: Echo task inst
     schedule:
       type: random
       interval: 1s
     args:
       name-int64: -128
       name-int64-2: 127
   task-inst:
     task-name: sink-graphite
     task-inst-name: Graphite sink inst
     schedule:
       type: triggered-raw
       task-name: echo
     args:
       server: localhost:2013
       queue-size: 1000

The config file has two types of top-level elemets: ``modules`` and ``task-inst``. The ``task-inst`` element may be repeated multiple times, and each ``task-inst`` contains configuration for a single task instance. The ``modules`` element appears only once, and contains a single ``module-name`` element per module.

Here is the ``modules`` section highlighted:

.. code-block:: yaml
   :linenos:
   :emphasize-lines: 1-4

   modules:
     module-name: bitd-echo
     module-name: bitd-sink-graphite
     module-name: bitd-config-log
   task-inst:
     task-name: config-log
     task-inst-name: config-log
     #...

The ``module-names`` get translated into DLL names, and the DLLs are loaded into the running ``bitd-agent``. Different platforms have different naming conventions for the module DLLs. The DLL name corresponding to module-name ``abc`` is:

- ``libabc.so`` on Unix

- ``abc.dll`` on Win32 Windows

- ``cygabc.dll`` on Cygwin Windows

- ``libabc.dylib`` on OSX

These DLLs must be found in the platform-specific ``PATH``, or must be located in a folder specified using the ``-lp`` command line parameter to ``bitd-agent`` (see :ref:`load-path-parameter`). Each DLL will implement one or more tasks in the configuration.

Each ``task-inst`` block has one ``task-name`` element (a task implemented in one of the modules), and a ``task-inst-name`` that should be unique per task. Here are the task names and task instance names highlighted:

.. code-block:: yaml
   :linenos:
   :emphasize-lines: 6-7,16-17,25-26

   modules:
     module-name: bitd-echo
     module-name: bitd-sink-graphite
     module-name: bitd-config-log
   task-inst:
     task-name: config-log
     task-inst-name: config-log
     schedule:
       type: config
     input:
       log-level: warn
       log-key:
         key-name: bitd-sink-graphite
         log-level: debug
   task-inst:
     task-name: echo
     task-inst-name: Echo task inst
     schedule:
       type: random
       interval: 1s
     args:
       name-int64: -128
       name-int64-2: 127
   task-inst:
     task-name: sink-graphite
     task-inst-name: Graphite sink inst
     schedule:
       type: triggered-raw
       task-name: echo
     args:
       server: localhost:2013
       queue-size: 1000

The task instances each have a schedule element. The schedule determines when the task instance will run. 

.. code-block:: yaml
   :linenos:
   :emphasize-lines: 8-9, 18-20,27-29

   modules:
     module-name: bitd-echo
     module-name: bitd-sink-graphite
     module-name: bitd-config-log
   task-inst:
     task-name: config-log
     task-inst-name: config-log
     schedule:
       type: config
     input:
       log-level: warn
       log-key:
         key-name: bitd-sink-graphite
         log-level: debug
   task-inst:
     task-name: echo
     task-inst-name: Echo task inst
     schedule:
       type: random
       interval: 1s
     args:
       name-int64: -128
       name-int64-2: 127
   task-inst:
     task-name: sink-graphite
     task-inst-name: Graphite sink inst
     schedule:
       type: triggered-raw
       task-name: echo
     args:
       server: localhost:2013
       queue-size: 1000


Task instance schedule types
----------------------------

The task instance schedule can be of one of these types:

- ``periodic:`` most tasks are periodic. You can specify the task instance run interval at which the task is running. The period is specified by the ``interval`` element, which is of the form ``[number]us|ms|s|m|h|d``, where ``us`` is microseconds, ``ms`` is milliseconds, ``s`` is seconds, ``m`` is minutes, ``h`` is hours, ``d`` is days.

- ``random:`` this is similar to ``periodic``, but the start time is random within the period. The ``interval`` element specifies the period duration from which the random start is picked.

- ``once:`` these tasks run only once

- ``config:`` these tasks run once when the ``bitd-agent`` is initialized, and also each time the config is reloaded. On Unix and OSX, the config is reloaded when the ``bitd-agent`` receives a ``SIGHUP`` signal. On Windows it's not possible to reload the config.

- ``triggered`` and ``triggered-raw:`` these tasks are triggered by other tasks in the config file, and take as input the output of these other tasks. When a task instance is ``triggered``, it gets only the output of the task instance run that triggered it. When a task instance has a ``triggered-raw`` schedule, it gets the ``tags``, ``run-id``, ``run-timestamp``, ``exit-code``, ``output`` and ``error`` of the task instance run that triggered it.


