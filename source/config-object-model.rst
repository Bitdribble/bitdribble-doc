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

bitd_void
---------

``bitd_void`` represents the empty type, and is defined as the ``C`` type ``void``.

bitd_boolean
------------
``bitd_boolean`` is defined as ``signed char``, and holds ``FALSE`` as ``0`` and ``TRUE`` as any non-zero value.

bitd_int64, bitd_uint64
-----------------------
``bitd_int64``, ``bitd_uint64`` are unsigned and signed 64 bit data types, defined as ``long long``, resp. ``unsigned long long`` on all platforms where ``long long`` is 64 bits.

The header file defines, in similar vein, data types for smaller width integers: ``bitd_int8``, ``bitd_uint8``, ``bitd_int16``, ``bitd_uint16``, ``bitd_int32``, ``bitd_uint32``. While these types are used frequently in the source code, the Bitdribble object model represents all integers parsed from ``yaml`` or ``xml`` as either ``bitd_int64`` or ``bitd_uint64``, for simplicity. 

bitd_double
-----------
``bitd_double`` is mapped to the ``C`` language ``double`` float type. No ``bitd_float`` type is defined - we use ``bitd_double`` instead.

The composite data types for ``C`` objects are:

bitd_string
-----------
``bitd_string`` is used for NULL-terminated ``char *`` strings.

bitd_blob
---------
``bitd_blob`` holds arbitray buffers that may contain the character ``0``. The ``bitd_blob`` contains a 4 byte length field followed by the actual blob payload. 

.. code-block:: c++

   /* The blob type */
   typedef struct {
       bitd_uint32 nbytes;
   } bitd_blob;

   #define bitd_blob_size(b) ((b)->nbytes)
   #define bitd_blob_payload(b) ((char *)(((bitd_blob *)b)+1))

Nvp arrays
----------
``bitd_nvp_t`` holds an array of name-value pairs, each element of which has its own type. In short, this type is called an ``nvp`` array. Elements in an ``nvp`` array can have any simple or composite type, including the ``nvp`` array type itself.

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

Objects
-------
The ``bitd_object_t`` type holds any arbitrary typed value:
 
.. code-block:: c++

   /* An object is a typed value */
   typedef struct {
       bitd_value_t v;
       bitd_type_t type;
   } bitd_object_t;

Any object, thus, can be represented as ``bitd_object_t``. This means objects can be ``bitd_boolean``, or ``bitd_int64``, or of ``nvp`` type. And, since ``nvp``` is an array type, the objects of type ``nvp`` can be thought of as arrays of other objects.


The Yaml object model
=====================
For a quick introduction to ``yaml``, see https://en.wikipedia.org/wiki/YAML. Simple bitdribble types are represented in ``yaml`` as follows:

bitd_void
---------
``bitd_void`` is formatted as the empty ``yaml`` string. An empty ``yaml`` string is represented as a ``bitd_void`` type.

bitd_boolean
------------
``bitd_boolean`` is represented as the ``yaml`` string ``TRUE`` or ``FALSE``. The ``yaml`` strings ``TRUE`` and ``FALSE`` are represented as ``bitd_boolean``.

bitd_int64 and bitd_uint64
--------------------------
``bitd_int64`` and ``bitd_uint64`` are represented in ``yaml`` as numeric strings. Integer in ``yaml`` are represented as ``bitd_int64``, if between ``LLONG_MIN`` and ``LLONG_MAX``, and ``bitd_uint64`` if between ``LLONG_MAX+1`` and ``ULLONG_MAX``.

bitd_double
-----------
``bitd_double`` is represented in ``yaml`` as a numeric string formatted as a floating point number, in decimal format. Numeric strings in ``yaml`` that are not integers, or are outside of the ``int64`` and ``uint64`` range are represented in ``C`` as ``bitd_double``.

Composite bitdribble types are represented in ``yaml`` as follows:

bitd_string
-----------
``bitd_string`` is represented in ``yaml`` as a string. Yaml strings that are non-void, non-numeric, and not ``TRUE`` or ``FALSE`` are represemted in ``C`` as ``bitd_string`` types.

bitd_blob
---------
``bitd_blob`` types are represented in ``yaml`` as ``base64`` encoded ``!!binary`` types. Conversely, ``!!binary`` yaml types are ``base64`` decoded and represented in ``C`` as ``bitd_blob`` types.

Nvp arrays
----------
``bitd_nvp_t`` types are represented in ``yaml`` as non-scalar name-value pairs. ``Nvp`` arrays with all elements having NULL names are represented as ``yaml`` sequences. Conversely, ``yaml`` composite types are represented as ``nvp`` arrays, and ``yaml`` sequences are represented as nvp arrays with NULL-named elements.

Objects
-------
The ``bitd_object_t`` type is represented in ``yaml`` simply by representing the underlying type and value of the object in ``yaml``. Conversely, a ``yaml`` document is represented by a ``bitd_object_t`` type.

This sets a correspondence between objects and ``yaml`` documents that is *onto*, in a mathematical sense: any ``yaml`` document corresponds to one or more objects. To see why this correspondence is not also *one to one*, observe that objects containing a string that is an integer corresponds to a ``yaml`` document containing that number's value, which in turn corresponds to an object of integer type.

``Yaml`` files can also contain a stream of documents. For example, the task instance results output of the ``bitd-agent`` is a ``yaml`` stream, with each task instance result being its own document. A ``yaml`` stream corresponds to an ordered set of ``C`` objects.

Using Yaml attributes
---------------------
As seen above, ``yaml`` strings are parsed into ``bitd_void`` if empty, or into ``bitd_boolean`` if equal to ``TRUE`` or ``FALSE``, or into ``bitd_int64`` if integers within the ``LLONG_MIN`` and ``LLONG_MAX``, or otherwise into ``bitd_uint64`` if between ``LLONG_MAX+1`` and ``ULLONG_MAX``, or otherwise into ``bitd_double`` if numeric - or, if none of the above, they are parsed as ``bitd_string``.

This represents the default conversion of ``yaml`` string scalars. The conversion can also be controlled by use of the following ``yaml`` attributes:

- ``tag:yaml.org,2002:null`` is converted to ``bitd_void`` type.

- ``tag:yaml.org,2002:bool`` is converted to ``bitd_boolean`` type.

- ``tag:yaml.org,2002:int`` is converted to ``bitd_int64``. The value is truncated if too large.

- ``tag:yaml.org,2002:str`` is converted to ``bitd_string``.

- ``tag:yaml.org,2002:binary`` is converted to ``bitd_blob``.

The source code
---------------
The implementation of the ``yaml`` object model is in src/libs/bitd/types-yaml.c.

The Xml object model
====================
For an introduction to ``xml``, see https://en.wikipedia.org/wiki/XML. For a quick introduction, ``xml`` documents emply angle brackets to delineate element names and element content. Elements can have zero or more attributes:

.. code-block:: xml

   <?xml version='1.0'?>
   <root-element-name>
     <element-name1/>
     <element-name2>127</element-name2>
     <element-name3 attribute1="value1" attribute2="value2">abc</element-name3>
     <element-name4>
       <embedded-element-name5 attribute1="value1">def</embedded-element-name5>
     </element-name4>
   </root-element-name>

The order of attributes is not important in an element, but the order of subelements matters - in the sense that changing the attribute order does not change the ``xml`` document, but changing the element order does change the ``xml`` document.

We will describe a partial correspondence between ``xml`` documents and *named* bitdribble objects. The ``root-element-name`` corresponds to the *name* of the ``object``. Each ``element-name`` corresponds to the ``name`` of a value in an ``nvp`` name-value pair array. If no attribute is specified, the type of the content is inferred:

- If the element is empty, the type is ``bitd_void``.

- If the element is the string ``TRUE`` or ``FALSE``, the type is ``bitd_boolean``.

- If the element is numeric string, the type is ``bitd_int64`` if an integer between ``LLONG_MIN`` and ``LLONG_MAX``, otherwise ``bitd_uint64`` if an integer between ``LLONG_MAX+1`` and ``ULLONG_MAX``, and otherwise a ``bitd_double``.

- If the element is any other string, the type is ``bitd_string``.

- If the element has sub-elements, the type is ``bitd_nvp_t``.

Using specific ``xml`` attributes changes the type of the element:

- If the element has an attribute named ``type`` with value ``void``, respectively ``boolean``, ``int64``, ``uint64``, ``double``, ``string``,   the type is ``bitd_void``, respectively ``bitd_boolean``, ``bitd_int64``, ``bitd_uint64``, ``bitd_double``, ``bitd_string``.

- If the element has the attribute ``type='blob'``, the value is interpreted to be a base64 encoded ``bitd_blob``.

- If the element has the attribute ``type='nvp'``, the value is interpreted to be of type ``bitd_nvp_t``.

The converse correspondence is described below:

bitd_void
---------
``bitd_void`` types are represented as empty ``xml`` elements. Optionally, these elements can be assigned a ``type='void'`` attribute.

Only ``xml`` documents with specific attributes have a named ``object`` correspondent.

     
A Bitdribble object is converted into an XML document.  Simple bitdribble types are represented in ``yaml`` as follows:

