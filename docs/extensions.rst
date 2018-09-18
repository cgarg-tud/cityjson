==========
Extensions
==========

.. contents:: :local:

The CityGML data model allows us to represent the most common city features, but sometimes practitioners may want to model additional features and/or add certain attributes to the data model.
For this, CityGML has the concept of `ADEs (Application Domain Extensions) <https://www.citygml.org/ade/>`_.
An ADE is defined in an extra `XML Schema <https://en.wikipedia.org/wiki/XML_schema/>`_ (XSD file) with its own namespace, and often inheritance is used to refine the classes of the CityGML data model, to define entirely new classes, and to modify any class by adding for instance new geometries and complex attribute anywhere in a City Model.
The ADE allows us to document in a structured way, and also to validate, an instance of a CityGML document that would contain both classes from the core model and from the ADEs.

-------------------
CityJSON Extensions
-------------------

CityJSON uses `JSON Schemas <http://json-schema.org/>`_ to document and validate the data model, and these are less flexible than XML Schemas.
Inheritance and namespaces are for instance not supported; schemas should be seen as basically validating the syntax of a JSON document.

CityJSON nevertheless allows its data model to be extended with what is called *Extensions*.
These Extensions are defined as JSON Schemas, and thus their content can be validated.
The following cases are possible with CityJSON extensions:

  1. Adding new (complex) attributes to City Objects
  2. Creating a new City Object, or "extending" one, and define complex geometries

.. important::

  While Extensions are less flexible than CityGML ADEs (ie they have a narrower scope and less customisation is possible), it should be noted that the flexibility of ADEs come at a price: standard software (eg viewer or spatial analysis software) will not process correctly the files containing ADEs since specific code needs to be written. CityJSON Extensions are designed as such that they can be read and processed by standard CityJSON software, often no changes in the code is required. This is achieved by enforcing a set of simple rules, as defined below, when adding new complex attributes and City Objects; if these are followed then a CityJSON files containing Extensions will be seen as 'standard' CityJSON files.


1. Adding new (complex) attributes to City Objects
**************************************************

One of the philosophy of JSON is "schema-less", which means that one is allowed to define "new" properties for the JSON objects.
While this is in contrast to CityGML (and GML as a whole where schemas are encouraged), the schemas of CityJSON (:doc:`schema`) are partly following that philosophy.
That is, for a given City Object, the list of "allowed" properties/attributes is listed in the schema, but it is not an error to add new ones. 
The validator of CityJSON (`cjio <https://github.com/tudelft3d/cjio>`_ with the option ``--validate``) does more than simply validate a dataset against the schemas, and will return a *warning* if an attribute is not in the schema, but it is not considered invalid in CityJSON.

In brief, if one wants to add a new attribute to a given ``"Building"``, say to document its colour (``"colour": "red"``), the easiest way is to add a property to the City Object:

.. code-block:: js

  {
    "type": "Building", 
    "bbox": [ 84710.1, 446846.0, -5.3, 84757.1, 446944.0, 40.9 ],
    "attributes": { 
      "roofType": "gable",
      "colour": "red"
    },
    "geometry": [...]
  }

Observe that complex attributes (ie 'hierarchical') can be added:

.. code-block:: js

  {
    "type": "Building", 
    "bbox": [ 84710.1, 446846.0, -5.3, 84757.1, 446944.0, 40.9 ],
    "attributes": { 
      "roofType": "gable",
      "colour": {
        "facade": {
          "rgba": [255,255,255,1],
          "hex": "#000"
        },
        "roof": {
          "rgba": [0,255,0,1],
          "hex": "#0F0"
        }
      }
    },
    "geometry": [...]
  }

However, these will not be documented, nor will they be validated.
It is recommended to document complex attributes in a JSON schema, and thus a new City Object needs to be defined (as explained below).


2. Creating/extending new City Objects
**************************************

The creation of a new City Object is done by defining it in a JSON schema file.
Since all City Objects are documented in the schemas of CityJSON (in `cityobjects.json <https://github.com/tudelft3d/cityjson/blob/master/schema/v07/cityobjects.json>`_), it is basically a matter of copying the parts needed in a new file and modifying its content.
A new name for the City Object (for the class) must be given.
  
It should be observed that since JSON schema does not allow inheritance, the only way to extend a City Object is to define an entirely new one (with a new name, eg ``"+NoiseBuilding"``).
This is done by copying the schema of the parent City Object and extending it. 

.. important::

  The challenge is creating Extensions that will not break the software packages (viewers, spatial analysis, etc) that already read and process CityJSON.
  While one could define a new City Object and document it, if this new object doesn't follow the rules below then it will mean that new specific software needs to be built for it; this would go against the fundamental ideas behind CityJSON.


When defining new City Objects, the following rules should be followed:

  1. The name of a new City Object must begin with a ``+``, eg ``"+NoiseBuilding"``
  2. A new City Object must conform to the rules of CityJSON, ie it must contain a property ``"type"`` and one ``"geometry"``. If the object contains appearances, the same schemes should be used so that the new City Objects can be processed by the tools without modification. 
  3. All the geometries must be in the property ``"geometry"``, and cannot be located somewhere else deep in a hierarchy of a new property. This ensures that all the code written to process, manipulate, and view CityJSON files (eg `cjio <https://github.com/tudelft3d/cjio>`_ and `azul <https://github.com/tudelft3d/azul>`_) will be working without modifications. 
  4. If a new City Object needs to store more geometries (see below for an example), then a new City Object needs to be defined using the same structure of parent-children, as used by ``"Building"`` and ``"BuildingPart"``.
  5. The reuse of types defined in CityJSON, eg ``"Solid"`` or semantic surfaces, is allowed.
  6. To define new semantic surfaces, simply add a ``+`` to its name, eg ``"+ThermalSurface"``.

  
If a CityJSON file contains City Objects not in the core, then the CityJSON must contain an extra member called ``"extensions"`` whose values are the name-value pairs of the new City Objects and the name of the file (this can be a URI where the schema is hosted).

.. code-block:: js

  {
    "type": "CityJSON",
    "version": "0.7",
    "extensions": {
      "+TallBuilding": "https://www.hugo.com/extensions/improved_buildings.json",
      "+Statue": "https://www.hugo.com/extensions/statues.json"
    },
    "CityObjects": {},
    "vertices": []
  }


------------------------------------------------
Mapping of the Noise ADE to a CityJSON Extension
------------------------------------------------

To illustrate the process of creating a new CityJSON extension, we use the Noise ADE, which is the example case in the `CityGML 2.0 documentation <https://portal.opengeospatial.org/files/?artifact_id=47842>`_ (Section 10.13.2 on p. 151 describes it; and Annex H on p. 305 gives more implementation details).
The XSDs and some test datasets are available `here <http://schemas.opengis.net/citygml/examples/2.0/ade/noise-ade/>`_.

The resulting files for the Noise Extension are available:
  - :download:`download e_noise.json <../extensions/e_noise.json>`
  - :download:`download noise_data.json <../example-datasets/extensions/noise_data.json>`


Adding new attributes to a Building
***********************************

.. image:: _static/noise_building.png
   :width: 60%

To add these attributes (they are not complex, but for the sake of the exercise let us assume that they are) one needs to:

  1. Define in a new schema file two new City Objects: ``"+NoiseBuilding"`` and ``"+NoiseBuildingPart"`` 
  2. Copy the schemas of ``"Building"`` and ``"BuildingPart"``, `defined in this file <https://github.com/tudelft3d/cityjson/blob/master/schema/v07/cityobjects.json>`_
  3. Extend these schemas and add a new property ``"noise-attributes"``. The new attributes could have been simply added to the list of ``"attributes"`` too.


.. code-block:: js

  "+NoiseBuilding": {
      "type": "object",
      "properties": {
        "type": { "enum": ["+NoiseBuilding"] },
        "attributes": ...
        "noise-attributes": {
          "buildingReflection": {"type": "string"},
          "buildingReflectionCorrection": {"type": "number"},
          "buildingLDenMax": {"type": "number"},
          "buildingLDenMin": {"type": "number"},
          "buildingLNightMax": {"type": "number"},
          "buildingLNightMin": {"type": "number"},
          "buildingLDenEq": {"type": "number"},
          "buildingLNightEq": {"type": "number"},
          "buildingHabitants": {"type": "integer"},
          "buildingImmissionPoints": {"type": "integer"},
          "remark": {"type": "string"}
        }
        ...


A CityJSON file containing this new City Object would look like this:

.. code-block:: js

  {
    "type": "CityJSON",
    "version": "0.7",
    "extensions": {
      "+NoiseBuilding": "e_noise.json" 
    },
    "CityObjects": {
      "1234": {
        "type": "+NoiseBuilding",
        "geometry": [
          {
            "type": "Solid",
            "lod": 2,
            "boundaries": [
              [ [[0, 3, 2, 1]], [[4, 5, 6, 7]], [[0, 1, 5, 4]], [[1, 2, 6, 5]], [[2, 3, 7, 6]], [[3, 0, 4, 7]] ] 
            ]
          }
        ],
        "attributes": {
          "roofType": "pointy"
        },
        "noise-attributes": {
          "buildingReflectionCorrection": 234,
          "buildingLNightMax": 17.33
        }
      },


Adding complex types for CityFurniture
**************************************

.. image:: _static/noise_cf.png
   :width: 80%

As it can be seen in the UML diagram, extending ``"CityFurniture"`` is more challenging because not only new simple attributes need to be defined, but a ``"CityFurniture"`` object can contain several ``"NoiseCityFurnitureSegment"``, which have their own geometry (a 'gml:Curve'). 


The steps to follow are thus:

  1. Create 2 new City Objects: ``"+NoiseCityFurniture"`` and ``"+NoiseCityFurnitureSegment"``
  2. ``"+NoiseCityFurniture"`` can be copied from ``"CityFurniture"``, and we need to add a new property ``"children"`` which contains a list of the IDs of the segments. This is similar to what is done for ``"BuildingParts"`` and ``"BuildingIntallations"``: each City Object has its own geometries, and they are linked together with this simple method.
  3. ``"+NoiseCityFurnitureSegment"`` is a new City Object and it gets the attributes common to all City Objects, and its geometry is restricted to a ``"MultiLineString"``. It also gets one property ``"parent"`` which links to its parent ``"+NoiseCityFurniture"``.

.. code-block:: js

  "+NoiseCityFurniture": {
    "type": "object",
    "properties": {
      "type": { "enum": ["+NoiseCityFurniture"] },
      ...
      "children": {
        "type": "array",
        "description": "the IDs of the +NoiseCityFurnitureSegment",
        "items": {"type": "string"}
      }
      ...
  }

.. code-block:: js

  "+NoiseCityFurnitureSegment": {
    "type": "object",
    "properties": {
      "type": { "enum": ["+NoiseCityFurnitureSegment"] },
      "attributes": {
        ...
      },
      "parent": { "type": "string" },
      "geometry": {
        "type": "array",
        "items": {
          "oneOf": [
            {"$ref": "geomprimitives.json#/MultiLineString"}
          ]
        }
      }
    },
    "required": ["type", "geometry", "parent"],
    "additionalProperties": false
  }


.. code-block:: js

  "a_noisy_bench": {
    "type": "+NoiseCityFurniture",
    "geometry": [
      {
        "type": "Solid",
        "lod": 2,
        "boundaries": [
          [ [[0, 3, 2, 1]], [[4, 5, 6, 7]], [[0, 1, 5, 4]], [[1, 2, 6, 5]], [[2, 3, 7, 6]], [[3, 0, 4, 7]] ] 
        ]
      }
    ],
    "children": ["thesegment_1", "thesegment_2"]
  },
  "thesegment_1": {
    "type": "+NoiseCityFurnitureSegment",
    "geometry": [
      {
        "type": "MultiLineString",
        "lod": 0,
        "boundaries": [
          [2, 3, 5], [77, 55, 212]
        ]
      }      
    ],
    "parent": "a_noisy_bench",
    "attributes": {
      "reflectionCorrection": 2.33
    }
  }    


-----------------------------------------
Validation of files containing extensions
-----------------------------------------

The validation of a CityJSON file containing extensions needs to be performed as a 2-step operation:
  1. The standard validation of all City Objects (except the new ones; those starting with ``"+"`` are ignored at this step); 
  2. Each new City Object is (individually) validated against its schema defined in the new schema file.

While this could be done with any JSON schema validator, resolving all the JSON references could be slightly tricky. 
Thus, `cjio <https://github.com/tudelft3d/cjio>`_ (with the option ``--validate``) has automated this process:

  - copy all the CityJSON schemas in a given folder (say ``/home/elvis/noise_extension/``), 
  - add your new schema to this folder (**important**, all schemas need to be in the same folder!)
  - and then:

.. code-block:: bash

  $ cjio noise_data.json validate --extensions --folder_schemas /home/elvis/noise_extension/


