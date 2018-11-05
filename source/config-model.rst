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

*******************
Configuration model
*******************

.. index::
   single: Configuration model

.. contents:: Table of Contents

This section describes configuration files, how they are structured, and how they are represented in ``C`` structures. The ``bitd-agent`` uses a common object model to describe all types of configuration. It is easier to understand the object model starting with its ``C`` language representation, then understand how it is translated into and from ``yaml`` and ``xml``. 

The ``bitd-agent`` configuration file is passed in using the ``-c`` argument. The configuration file can be formatted as either ``yaml`` or ``xml``. The ``bitd-agent`` uses utilities in the ``libbitd`` library to parse the configuration into ``C`` data types, then runs the configured task instances, and converts back the output of task instances from ``C`` data type format to ``yaml`` or ``xml``. 

Understanding the conversion mechanism between ``yaml``, ``xml`` and ``C`` data types, thus, facilitates working with the ``bitd-agent``.

The C object model
==================
Look at the ``include/bitd/platform-types`` header for the data types defined below. The primary data types for ``C`` objects are:

- ``bitd_void`` - the empty type.

- ``bitd_boolean`` - defined as ``signed char``, holding ``FALSE`` as ``0`` and ``TRUE`` as any non-zero value.

- ``bitd-int64``, ``bitd-uint64`` - the unsigned and signed 64 bit data types, defined as ``long long``, resp. ``unsigned long long`` on all platforms where ``long long`` is 64 bits.

- ``bitd-double`` - the double float type.

The header file defines, in similar vein, data types for smaller width integers: ``bitd-int8``, ``bitd-uint8``, ``bitd-int16``, ``bitd-uint16``, ``bitd-int32``, ``bitd-uint32``. While these types are used frequently in the source code, the Bitdribble object model represents all integers as either ``bitd-int64`` or ``bitd-uint64``, for simplicity.

The composite data types for ``C`` objects are:

- ``bitd-string``, for NULL-terminated ``char *`` strings.

- ``bitd-blob``, which holds arbitray buffers that may contain the character ``0``. 
