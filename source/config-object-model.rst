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
Look at the ``include/bitd/platform-types.h`` header for the data types defined below. The primary ``C`` data types are:

- ``bitd_void`` - the empty type.

- ``bitd_boolean`` - defined as ``signed char``, holding ``FALSE`` as ``0`` and ``TRUE`` as any non-zero value.

- ``bitd_int64``, ``bitd_uint64`` - the unsigned and signed 64 bit data types, defined as ``long long``, resp. ``unsigned long long`` on all platforms where ``long long`` is 64 bits.

- ``bitd_double`` - the double float type.

The header file defines, in similar vein, data types for smaller width integers: ``bitd_int8``, ``bitd_uint8``, ``bitd_int16``, ``bitd_uint16``, ``bitd_int32``, ``bitd_uint32``. While these types are used frequently in the source code, the Bitdribble object model represents all integers parsed from ``yaml`` or ``xml`` as either ``bitd_int64`` or ``bitd_uint64``, for simplicity. No ``bitd_float`` type is defined - we use ``bitd_double`` instead.

The composite data types for ``C`` objects are:

- ``bitd_string``, for NULL-terminated ``char *`` strings.

- ``bitd_blob``, which holds arbitray buffers that may contain the character ``0``. The ``bitd_blob`` contains a 4 byte length field followed by the actual blob payload. 

.. code-block:: c++

   /* The blob type */
   typedef struct {
       bitd_uint32 nbytes;
   } bitd_blob;

   #define bitd_blob_size(b) ((b)->nbytes)
   #define bitd_blob_payload(b) ((char *)(((bitd_blob *)b)+1))

Another composite type is:
 
- ``bitd_nvp_t``, which holds an array of name-value pairs, each element of which has its own type. In short, this type is called an ``nvp``. Elements in an ``nvp`` can have any simple or composite type, including the ``nvp`` type itself.

.. code-block:: c++

   /* Forward declaration */
   struct bitd_nvp_s;

   /* Enumeration of types */
   typedef enum {
       bitd_type_void,
       bitd_type_boolean,
       bitd_type_int64,
       bitd_type_uint64,
       bitd_type_double,
       bitd_type_string,
       bitd_type_blob,
       bitd_type_nvp,
       bitd_type_max
   } bitd_type_t;

   /* Untyped value */
   typedef union {
       bitd_boolean value_boolean;
       bitd_int64 value_int64;
       bitd_uint64 value_uint64;
       bitd_double value_double;
       bitd_string value_string;
       bitd_blob *value_blob;
       struct bitd_nvp_s *value_nvp;
   } bitd_value_t;

   /* A name-value-pair element - or 'nvp element' */
   typedef struct {
       char *name; 
       bitd_value_t v;
       bitd_type_t type;
   } bitd_nvp_element_t;

   /* A name-value-pair array - or 'nvp' */
   typedef struct bitd_nvp_s {
       int n_elts;
       int n_elts_allocated;
       bitd_nvp_element_t e[1]; /* Array of named objects */
   } *bitd_nvp_t;

The last composite type is:

- ``bitd_object_t``, which holds any arbitrary typed value:
 
.. code-block:: c++

   /* An object is a typed value */
   typedef struct {
       bitd_value_t v;
       bitd_type_t type;
   } bitd_object_t;

Any object, thus, can be represented as ``bitd_object_t``. This means objects can be ``bitd_boolean``, or ``bitd_int64``, or of ``nvp`` type. And, since ``nvp``` is an array type, the objects of type ``nvp`` can be thought of as arrays of other objects.


The Yaml object model
=====================
For a quick introduction to ``yaml``, see https://en.wikipedia.org/wiki/YAML. Bitdribble ``c`` objects are represented as yaml as follows:

- ``bitd_void`` is formatted as the empty string. An empty ``yaml`` string is represented as a ``bitd_void`` type.

- ``bitd_boolean`` is represented as the string ``TRUE`` or ``FALSE``. The ``yaml`` string ``TRUE`` and ``FALSE`` are represented as ``bitd_boolean``.

- ``bitd_int64`` and ``bitd_uint64`` are represented in ``yaml`` as numeric strings. Integer in ``yaml`` are represented as ``bitd_int64``, if between ``LLONG_MIN`` and ``LLONG_MAX``, and ``bitd_uint64`` if between ``LLONG_MAX+1`` and ``ULLONG_MAX``.
