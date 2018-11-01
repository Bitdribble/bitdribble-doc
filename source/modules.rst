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

*******
Modules
*******

.. contents:: Table of Contents

The following sections detail the types of modules supported.

Config modules
==============
All config modules are of schedule type ``config``, which means that they are executed once on ``bitd-agent`` start, and are execute again each time the ``bitd-agent`` is reloading its configuration (for example, because it received a Unix ``SIGHUP`` signal, or because the ``bitd-agent`` is running under ``systemd`` control and the following command was executed: ``systemd reload bitd``).


bitd-config-log
---------------
The `bitd-config-log`` module controls the ``bitd-agent`` log level. It is configured through the following ``input`` parameters:

- ``log-level``, which controls the global log level. The syntax of the ``log-level`` is:

.. code-block:: none

   log-level: none|crit|error|info|warn|debug|trace

- ``log-key``, which can be specified multiple times, and controls a subsystem log level. The syntax of the ``log-key`` entry is:

.. code-block:: none

   log-key:
     key-name: <key name>
     log-level: none|crit|error|info|warn|debug|trace
   
Here is an example ``config-log`` configuration:


.. code-block:: none
   :linenos:

   modules:
     module-name: bitd-config-log
   task-inst:
     task-name: config-log
     task-inst-name: Config log task instance
     schedule:
       type: config
     input:
       log-level: info
       log-key:
         key-name: module-mgr
         log-level: trace
       log-key:
         key-name: module-agent
         log-level: none

This will set the ``module-mgr`` subsystem log level to ``trace``, the ``module-agent`` subsystem log level to ``none`` (disabling that subsystem log), and the global log level for all other log messages to ``info``.

Run modules
===========
All run modules can execute periodically (with fixed or random start time), or can execute once. 

bitd-echo module
----------------
The ``bitd-echo`` module supports the ``echo`` task.

echo task
^^^^^^^^^
The ``echo`` task echoes back its ``args`` as ``output``. If ``args`` is void, it will instead echo back its ``input`` as ``output``. Here is an example configuration:

.. code-block:: none
   :linenos:
   :emphasize-lines: 9-11

   modules:
     module-name: bitd-echo
   task-inst:
     task-name: echo
     task-inst-name: Echo task instance
     schedule:
       type: periodic
       interval: 1s
     args:
       name-int64: -128
       name-int64: 127

Here is the ``bitd-agent`` result output:

.. code-block:: none
   :linenos:
   :emphasize-lines: 9-11

   $ bitd-agent -c echo.yml 
   ---
   tags:
     task: echo
     task-instance: Echo task instance
   run-id: 0
   run-timestamp: 1539717016893523079
   exit-code: 0
   output:
     name-int64: -128
     name-int64: 127
   ---
   ^C...

The same output will be displayed if the configuration is changed to replace ``args`` with ``input``:

.. code-block:: none
   :linenos:
   :emphasize-lines: 9

   modules:
     module-name: bitd-echo
   task-inst:
     task-name: echo
     task-inst-name: Echo task instance
     schedule:
       type: periodic
       interval: 1s
     input:
       name-int64: -128
       name-int64: 127

If both ``args`` and ``input`` are specified, only ``args`` is echoed back as ``output``.

bitd-exec module
----------------
The ``bitd-exec`` module supports the ``exec`` task.

exec task
^^^^^^^^^
The ``exec`` task spawns a child process and reports as results its exit code, its ``stdout`` and ``stderr``. The following parameters can be configured:

* ``args`` holds the following subparameters:

  * ``command`` holds the executable command name and arguments. The command name and the arguments are space separated. This parameter is required.

  * ``command-tmo`` holds the maximum duration, in seconds, of the child process. If the child process does not exit within the configured ``command-tmo``, it is terminated. This parameter is optional, and can be useful to set a limit to the run duration of the child process.

  * ``input-type: {auto, string, blob, xml, yaml}`` configures whether the ``input`` parameter is formatted as string, blob, xml or yaml before being passed as ``stdin`` to the child process. If configured as ``auto``, the ``input`` will be formatted as ``yaml``, except if ``input`` is of ``blob`` type, in which case it is formatted as blob. The default value of the parameter is ``auto``. 

  * ``output-type: {auto, string, blob, xml, yaml}`` configures whether the ``stdout`` is parsed as string, blob, xml or yaml. If configured as ``auto``, the ``stdout`` is checked, in order, for ``xml``, ``yaml``, ``string`` content, and is displayed respectively as ``xml``, ``yaml``, ``string``, or, if none of the above applies, as ``blob``. The default value of the parameter is ``auto``. 

    The output is considered to be ``xml`` or ``yaml`` if the ``xml``, respectively the ``yaml`` parser encounters no error. It is considered to be ``string`` format if it contains no ``NULL`` characters aside from the ``NULL`` termination.

  * ``error-type: {auto, string, blob, xml, yaml}`` configures whether the ``stderr`` is parsed as ``string``, ``blob``, ``xml`` or ``yaml``. If configured as ``auto``, the ``stderr`` is checked, in order, for ``xml``, ``yaml``, ``string`` content, and is displayed respectively as ``xml``, ``yaml``, ``string``, or, if none of the above applies, as ``blob``. The default value of the parameter is ``auto``. 

* The ``input`` parameter is passed as standard input to the child process. The format of the ``stdin`` buffer is determined by the ``args.input-type`` parameter.

The task instance ``tags`` are passed down to the child process as environment variables. Recall that ``tags`` can be task instance scoped, module scoped, or globally scoped at the level of the configuration file. Tags from all three scopes are merged together, with module scope tags taking precedence over global scoped tags, and task instance scoped tags taking precedence over module scope tags. 

.. TO DO: explain conversion of tags to environment variables. Right now, only tags of string types with non-empty names are passed in as environment variables.

The shell child process will have an exit code, and will output ``stdout`` and ``stderr``:

* The exit code is reported back as the ``exit-code`` result.

* The ``stdout`` is reported back as the ``output`` result. The ``args.output-type`` parameter controls how ``stdout`` is parsed.

* The ``stderr`` is reported back as the ``error`` result. The ``args.error-type`` parameter controls how ``stdout`` is parsed.

The shell command is specified by the ``command`` subparameter of ``args``:

.. code-block:: none
   :linenos:
   :emphasize-lines: 10

   modules:
     module-name: bitd-exec
   task-inst:
     task-name: exec
     task-inst-name: Exec task instance
     schedule:
       type: periodic
       interval: 1s
     args:
       command: echo hi

Here is the output of this configuration file:

.. code-block:: none
   :linenos:
   :emphasize-lines: 8-9

   $ bitd-agent -c exec.yml 
   ---
   tags:
     task: echo
     task-instance: Exec task instance
   run-id: 0
   run-timestamp: 1539717825481153340
   exit-code: 0
   output: hi
   ---
   ^C...

Here is an example with output sent both to ``stdout`` and ``stderr``, and with a non-zero exit code:

.. code-block:: none
   :linenos:
   :emphasize-lines: 10

   modules:
     module-name: bitd-exec
   task-inst:
     task-name: exec
     task-inst-name: Exec task instance
     schedule:
       type: periodic
       interval: 1s
     args:
       command: echo hi; echo ho >&2; exit 3

And the output is:

.. code-block:: none
   :linenos:
   :emphasize-lines: 8-10

   $ bitd-agent -c exec.yml 
   ---
   tags:
     task: echo
     task-instance: Exec task instance
   run-id: 0
   run-timestamp: 1539718218405943930
   exit-code: 3
   output: hi
   error: ho
   ---
   ^C...


