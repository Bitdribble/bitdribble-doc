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

The Json object model
=====================
For the definition of ``json``, see https://www.json.org. Simple bitdribble types are represented in ``json`` as follows:

bitd_void
---------
``bitd_void`` is formatted as the ``null`` json value, and the ``null`` json value is represented as a ``bitd_void`` type.

bitd_boolean
------------
``bitd_boolean`` ``TRUE`` is represented in ``json`` as ``true``, and ``bitd_boolean`` ``FALSE`` as ``false``. Conversely, json ``true``, ``false`` are respectively represented as ``bitd_boolean`` values ``TRUE`` and ``FALSE``.

bitd_int64 and bitd_uint64
--------------------------
``bitd_int64`` and ``bitd_uint64`` are represented in ``json`` as numeric integers, except when the ``bitd_uint64`` value is larger than ``LONG_MAX``, in which case it is represented as a string. When ``bitd_uint64`` is represented as string, the key name is appended the suffix ``_!!uint64``.

Conversely, numeric ``json`` integers are represented as ``bitd_int64``, or if the key name has a ``_!!uint64`` suffix, as a ``bitd_uint64``.

bitd_double
-----------
``bitd_double`` is represented in ``json`` as a numeric string formatted as a floating point number, in decimal format. Numeric strings in ``json`` that have a floating point, or are outside of the ``int64`` and ``uint64`` range are represented in ``C`` as ``bitd_double``.

Composite bitdribble types are represented in ``json`` as follows:

bitd_string
-----------
``bitd_string`` is represented in ``json`` as a string. Json strings that are non-void, non-numeric, and do not have a key suffix of ``_!!uint64`` or ``_!!blob`` format are represented as ``bitd_string``. 

bitd_blob
---------
``bitd_blob`` types are represented in ``json`` as ``base64`` encoded strings, with a key name that gets appended a ``_!!blob`` suffix. Conversely, json values of string type with a key name suffix of ``_!!blob`` are ``base64`` decoded and represented as ``bitd_blob`` types.

Nvp arrays
----------
``bitd_nvp_t`` types are represented as ``json object``, if at least one of the element names are non-NULL and a non-zero-length string - and as a json ``array`` otherwise. A ``json object`` is represented as ``nvp`` array, and a ``json`` array is represented as nvp array with NULL-named elements.

Objects
-------
The ``bitd_object_t`` type is represented in ``json`` simply by representing the underlying type and value of the object in ``json``. Conversely, a ``json`` object is represented by a ``bitd_object_t`` type.

This sets a correspondence between ``bitd_object_t`` objects and ``json`` objects that is *onto*, in a mathematical sense: any ``json`` object corresponds to one or more ``bitd_object_t`` objects. 

Using Json attributes
---------------------
The ``bitd_object_t`` type can be implied from the ``json`` value type, but can also be set explicitly in the ``json`` object by appending ``_!!<type>`` to the key name corresponding to the ``json`` value. Here the ``<type>`` is any of ``void``, ``boolean``, ``int64``, ``uint64``, ``double``, ``string``, ``blob`` or ``nvp``.

The source code
---------------
The implementation of the ``json`` object model is in src/libs/bitd/types-json.c.

The Xml object model
====================
For an introduction to ``xml``, see https://en.wikipedia.org/wiki/XML. For a quick introduction, ``xml`` documents emply angle brackets to delineate element names and element content. Elements can have zero or more attributes:

.. code-block:: xml

   <?xml version='1.0'?>
   <root-element-name>
     <element-name1/>
     <element-name2>127</element-name2>
     <element-name3 attribute1='value1' attribute2='value2'>abc</element-name3>
     <element-name4>
       <embedded-element-name5 attribute1='value1'>def</embedded-element-name5>
     </element-name4>
   </root-element-name>

The order of attributes is not important in an element, but the order of subelements matters - in the sense that changing the attribute order does not change the ``xml`` document, but changing the element order does change the ``xml`` document.

We will describe a partial correspondence between ``xml`` documents and *named* bitdribble objects. The ``root-element-name`` corresponds to the *name* of the ``object``. Each ``element-name`` corresponds to the ``name`` of a value in an ``nvp`` name-value pair array. If no attribute is specified, the type of the content is inferred:

- If the element is empty, the type is ``bitd_void``.

- If the element is the string ``TRUE``, ``True``, ``true``, ``FALSE``, ``False`` or ``false``, the type is ``bitd_boolean``.

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

.. code-block:: xml

   <element-name/>
   <!-- or -->
   <element-name type='void'/>

bitd_boolean
------------
``bitd_boolean`` types are represented as ``xml`` elements having the ``TRUE`` or ``FALSE`` boolean value as contents. Optionally, these elements can be assigned a ``type='boolean'`` attribute.

.. code-block:: xml

   <element-name>FALSE</element-name>
   <!-- or -->
   <element-name type='boolean'>FALSE</element-name>

bitd_int64
------------
``bitd_int64`` types are represented as ``xml`` elements having as contents the integer value. Optionally, these elements can be assigned a ``type='int64'`` attribute. If the integer is between ``LLONG_MIN`` and ``LLONG_MAX``, the attribute can be omitted.

.. code-block:: xml

   <element-name>123</element-name>
   <!-- or -->
   <element-name type='int64'>123</element-name>

bitd_uint64
------------
``bitd_uint64`` types are also represented as ``xml`` elements having as contents the integer value. Optionally, these elements can be assigned a ``type='uint64'`` attribute. If the integer is between ``LLONG_MAX+1`` and ``ULLONG_MAX``, the attribute can be omitted.

.. code-block:: xml

   <element-name>18446744073709551615</element-name>
   <!-- or -->
   <element-name type='uint64'>123</element-name>

bitd_double
------------
``bitd_double`` types are represented as ``xml`` elements having as contents the numeric value. Optionally, these elements can be assigned a ``type='double'`` attribute. If the number has a decimal point or is not a ``bitd_int64`` or ``bitd_uint64``, the attribute can be omitted.

.. code-block:: xml

   <element-name>123.1</element-name>
   <!-- or -->
   <element-name>123.0</element-name>
   <!-- or -->
   <element-name type='double'>123</element-name>
   <!-- but not -->
   <element-name>123</element-name><!-- ...which would be interpreted as int64 -->
   
bitd_string
-----------
``bitd_string`` types are represented as ``xml`` elements having as contents the string value. Optionally, these elements can be assigned a ``type='string'`` attribute. If the value cannot be interpreted as a ``bitd_void``, ``bitd_boolean``, ``bitd_int64``, ``bitd_uint64``, ``bitd_double``, then the attribute can be omitted.

.. code-block:: xml

   <element-name>abc</element-name>
   <!-- or -->
   <element-name type='string'>abc</element-name>
   <!-- but not -->
   <element-name></element-name><!-- ...which would be interpreted as void -->
   <!-- and not -->
   <element-name>TRUE</element-name><!-- ...which would be interpreted as boolean -->
   <!-- and not -->
   <element-name>123</element-name><!-- ...which would be interpreted as int64 -->
   <!-- and not -->
   <element-name>123.0</element-name><!-- ...which would be interpreted as double -->
   
bitd_blob
---------
``bitd_blob`` types are represented as ``xml`` elements having as contents the *base64* encoded blob. These elements must be assigned a ``type='blob'`` attribute, to be distinguished from other strings. The attribute can never be omitted.

.. code-block:: xml

   <element-name type='blob'>MDEyMzQ1Njc4OQo=</element-name>

To find out to which blob contents this corresponds, you can uudecode it as follows:

.. code-block:: shell

   $ echo MDEyMzQ1Njc4OQo= | base64 -d
   0123456789
   
bitd_nvp
--------
``bitd_nvp`` types are name-value pair arrays and are represented as ``xml`` elements with subelements. Here is, for example, an ``nvp`` with elements of all possible types:

.. code-block:: xml

   <?xml version='1.0'?>
   <nvp type='nvp'>
     <name-void type='void'/>
     <name-boolean type='boolean'>FALSE</name-boolean>
     <name-int8 type='int64'>-128</name-int8>
     <name-int8 type='int64'>127</name-int8>
     <name-uint8 type='int64'>255</name-uint8>
     <name-int16 type='int64'>-32768</name-int16>
     <name-int16 type='int64'>32767</name-int16>
     <name-uint16 type='int64'>65535</name-uint16>
     <name-int32 type='int64'>-2147483648</name-int32>
     <name-int32 type='int64'>2147483647</name-int32>
     <name-uint32 type='int64'>4294967295</name-uint32>
     <name-int64 type='int64'>-9223372036854775808</name-int64>
     <name-int64 type='int64'>9223372036854775807</name-int64>
     <name-uint64 type='uint64'>18446744073709551615</name-uint64>
     <name-double type='double'>100000.0</name-double>
     <name-string type='string'/>
     <name-string type='string'>True</name-string>
     <name-string type='string'>123</name-string>
     <name-string type='string'>123.0</name-string>
     <name-string type='string'>string-value</name-string>
     <name-blob type='blob'>MDEyMzQ1Njc4OQo=</name-blob>
     <empty-nvp-value type='nvp'/>
     <full-nvp-value type='nvp'>
       <name-void type='void'/>
       <name-boolean type='boolean'>FALSE</name-boolean>
       <name-int8 type='int64'>-127</name-int8>
       <name-uint8 type='int64'>255</name-uint8>
       <name-int16 type='int64'>-32767</name-int16>
       <name-uint16 type='int64'>65535</name-uint16>
       <name-int32 type='int64'>-2147483647</name-int32>
       <name-uint32 type='int64'>4294967295</name-uint32>
       <name-int64 type='int64'>-9223372036854775807</name-int64>
       <name-uint64 type='uint64'>18446744073709551615</name-uint64>
       <name-double type='double'>1.99</name-double>
       <name-string type='string'/>
       <name-string type='string'>True</name-string>
       <name-string type='string'>123</name-string>
       <name-string type='string'>123.0</name-string>
       <name-string type='string'>string-value</name-string>
       <name-blob type='blob'>MDEyMzQ1Njc4OQo=</name-blob>
       <empty-nvp-value type='nvp'/>
     </full-nvp-value>
   </nvp>
   
If the nvp is named, the name will be stored as the ``xml`` root element name. If the nvp is unnamed, or has an empty name, by convention the root element name is set to ``nvp`` - as was the case in the example above. Here is the same ``xml`` document leaving out all ``type`` attributes that are optional (meaning that the type of the contents can be inferred from the value of the contents):

.. code-block:: xml

   <?xml version='1.0'?>
   <nvp>
     <name-void/>
     <name-boolean>FALSE</name-boolean>
     <name-int8>-128</name-int8>
     <name-int8>127</name-int8>
     <name-uint8>255</name-uint8>
     <name-int16>-32768</name-int16>
     <name-int16>32767</name-int16>
     <name-uint16>65535</name-uint16>
     <name-int32>-2147483648</name-int32>
     <name-int32>2147483647</name-int32>
     <name-uint32>4294967295</name-uint32>
     <name-int64>-9223372036854775808</name-int64>
     <name-int64>9223372036854775807</name-int64>
     <name-uint64>18446744073709551615</name-uint64>
     <name-double>100000.0</name-double>
     <name-string type='string'/>
     <name-string type='string'>True</name-string>
     <name-string type='string'>123</name-string>
     <name-string type='string'>123.0</name-string>
     <name-string>string-value</name-string>
     <name-blob type='blob'>MDEyMzQ1Njc4OQo=</name-blob>
     <empty-nvp-value type='nvp'/>
     <full-nvp-value>
       <name-void/>
       <name-boolean>FALSE</name-boolean>
       <name-int8>-127</name-int8>
       <name-uint8>255</name-uint8>
       <name-int16>-32767</name-int16>
       <name-uint16>65535</name-uint16>
       <name-int32>-2147483647</name-int32>
       <name-uint32>4294967295</name-uint32>
       <name-int64>-9223372036854775807</name-int64>
       <name-uint64>18446744073709551615</name-uint64>
       <name-double>1.99</name-double>
       <name-string type='string'/>
       <name-string type='string'>True</name-string>
       <name-string type='string'>123</name-string>
       <name-string type='string'>123.0</name-string>
       <name-string>string-value</name-string>
       <name-blob type='blob'>MDEyMzQ1Njc4OQo=</name-blob>
       <empty-nvp-value type='nvp'/>
     </full-nvp-value>
   </nvp>
   
The Yaml object model
=====================
For a quick introduction to ``yaml``, see https://en.wikipedia.org/wiki/YAML. Simple bitdribble types are represented in ``yaml`` as follows:

bitd_void
---------
``bitd_void`` is represented as the empty ``yaml`` string. An empty ``yaml`` string is represented as a ``bitd_void`` type.

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
``bitd_string`` is represented in ``yaml`` as a string. Yaml strings that are non-void, non-numeric, and not ``TRUE``, ``True``, ``true``, ``FALSE``, ``False`` or ``false`` are represented in ``C`` as ``bitd_string`` types.

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
As seen above, ``yaml`` strings are parsed into ``bitd_void`` if empty, or into ``bitd_boolean`` if equal to ``TRUE``, ``True``, ``true``, ``FALSE``, ``False`` or ``false``, or into ``bitd_int64`` if integers within the ``LLONG_MIN`` and ``LLONG_MAX``, or otherwise into ``bitd_uint64`` if between ``LLONG_MAX+1`` and ``ULLONG_MAX``, or otherwise into ``bitd_double`` if numeric - or, if none of the above, they are parsed as ``bitd_string``.

This represents the default conversion of ``yaml`` string scalars. The conversion can also be controlled by use of the following ``yaml`` attributes:

- ``tag:yaml.org,2002:null`` is converted to ``bitd_void`` type.

- ``tag:yaml.org,2002:bool`` is converted to ``bitd_boolean`` type.

- ``tag:yaml.org,2002:int`` is converted to ``bitd_int64``. The value is truncated if too large.

- ``tag:yaml.org,2002:str`` is converted to ``bitd_string``.

- ``tag:yaml.org,2002:binary`` is converted to ``bitd_blob``.

The source code
---------------
The implementation of the ``yaml`` object model is in src/libs/bitd/types-yaml.c.

