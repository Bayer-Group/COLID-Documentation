# Get Metadata & Instances

This section explains how to get and use meta data but also how to retrieve and use instances of data types from COLID. This data is needed to create a correct request for creating/editing COLID entries.

## API: Metadata

This API has two interfaces which gives one the ability to query the hierarchy of metadata concepts but also the metadata of a concrete COLID entry itself.

### The `metadata/hierarchy` interface

This interfaces delivers the hierarchy of metadata concepts:

```shell
GET "https://localhost:51770/api/v3/metadata/hierarchy"
```

The response lists all metadata concepts, it´s inheritances and so also all COLID entry types. When seen as a tree structure the COLID entry types are the leave nodes:

```json
{
  "key": "https://pid.bayer.com/kos/19050/PID_Concept",
  "label": "COLID entry",
  "description": "This is the artificial starting concept for the COLID application.",
  "subClasses": [
    {
      "key": "https://pid.bayer.com/kos/19050/MathematicalModel",
      "label": "Mathematical Model",
      "description": "This concept represents models which have been created by data scientists.",
      "subClasses": []
    },
    {
      "key": "https://pid.bayer.com/kos/19050/NonRDFDataset",
      "label": "Dataset",
      "description": "Any resource that contains instance data, e.g. CSV, database table, exposed view as well as RDF instances.",
      "subClasses": [
        {
          "key": "https://pid.bayer.com/d188c668-b710-45b2-9631-faf29e85ac8d/RWD_Source",
          "label": "RWD Source"
          "description": "routinely collected data relating to a patient&rsquo;s health status or the delivery of health care from a variety of sources other than traditional clinical trials https://ascpt.onlinelibrary.wiley.com/doi/epdf/10.1002/cpt.1426",
          "subClasses": []
        },
        {
          "key": "https://pid.bayer.com/kos/19050/GenericDataset",
          "label": "Generic Dataset",
          "description": "Any dataset that is not an RDF dataset. It can be a file like xml, xls, csv or data stored in systems like sql, MongoDB, Neo4J etc.",
          "subClasses": []
        },
...
```

It is important to know that this hierarchy might change over time, caused e.g. by additional entry types!

### The `metadata` interface

This interfaces delivers the metadata of a concrete concept.
In this example the COLID entry type `https://pid.bayer.com/kos/19050/GenericDataset` was chosen:

```shell
GET "https://localhost:51770/api/v3/metadata?entityType=https%3A%2F%2Fpid.bayer.com%2Fkos%2F19050%2FGenericDataset"
```

The response lists all properties with their name and a textual description but also with additional metadata like the datatype, the number of appearances or a regex condition of the value. Some properties are described in [SHACL](https://www.w3.org/TR/shacl/) notation:

```json
[
  ...
  {
    "key": "https://pid.bayer.com/kos/19050/hasVersion",
    "properties": {
    "http://pid.bayer.com/kos/19014/hasPID": "https://pid.bayer.com/kos/19050/hasVersion",
    "http://www.w3.org/ns/shacl#group": {
      "key": "https://pid.bayer.com/kos/19050/UsageAndMaintenance",
        "label": "Usage and Maintenance",
        "order": 6,
        "editDescription": "Grouping all Information for Usage and Maintenance",
        "viewDescription": "Grouping all Information for Usage and Maintenance"
      },
    "http://www.w3.org/ns/shacl#maxCount": "1",
    "http://www.w3.org/ns/shacl#minCount": "1",
    "http://www.w3.org/ns/shacl#path": "https://pid.bayer.com/kos/19050/hasVersion",
    "http://www.w3.org/ns/shacl#order": "4",
    "http://www.w3.org/ns/shacl#pattern": "^(\\d+\\.)*\\d+$",
    "http://www.w3.org/ns/shacl#nodeKind": "http://www.w3.org/ns/shacl#Literal",
    "http://www.w3.org/ns/shacl#name": "Version",
    "http://www.w3.org/2000/01/rdf-schema#label": "has version",
    "http://www.w3.org/ns/shacl#datatype": "http://www.w3.org/2001/XMLSchema#string",
    "http://www.w3.org/1999/02/22-rdf-syntax-ns#type": "http://www.w3.org/2002/07/owl#DatatypeProperty",
    "http://www.w3.org/2000/01/rdf-schema#comment": "This is the version of the resource that has been registered in COLID.",
    "http://www.w3.org/2000/01/rdf-schema#domain": "https://pid.bayer.com/kos/19050/PID_Concept",
    "http://www.w3.org/2000/01/rdf-schema#range": "http://www.w3.org/2001/XMLSchema#float"
  },
  nestedMetadata []
  },
  ...
  {
    "key": "https://pid.bayer.com/kos/19050/hasEntryLifecycleStatus",
    "properties": {
      "http://pid.bayer.com/kos/19014/hasPID": "https://pid.bayer.com/kos/19050/hasEntryLifecycleStatus",
      "http://www.w3.org/ns/shacl#group": {
        "key": "https://pid.bayer.com/kos/19050/TechnicalInformation",
        "label": "Technical Information",
        "order": 998,
        "editDescription": "Grouping all not editable technical Information",
        "viewDescription": "Grouping all not editable technical Information"
      },
      "http://www.w3.org/ns/shacl#maxCount": "1",
      "http://www.w3.org/ns/shacl#minCount": "1",
      "http://www.w3.org/ns/shacl#order": "1",
      "http://www.w3.org/ns/shacl#path": "https://pid.bayer.com/kos/19050/hasEntryLifecycleStatus",
      "http://www.w3.org/1999/02/22-rdf-syntax-ns#type": "http://www.w3.org/2002/07/owl#ObjectProperty",
      "http://www.w3.org/ns/shacl#name": "Entry Lifecycle Status",
      "http://www.w3.org/ns/shacl#nodeKind": "http://www.w3.org/ns/shacl#IRI",
      "http://www.w3.org/2000/01/rdf-schema#label": "hasEntryLifecycleStatus",
      "http://www.w3.org/2000/01/rdf-schema#domain": "https://pid.bayer.com/kos/19050/PID_Concept",
      "http://www.w3.org/2000/01/rdf-schema#range": "https://pid.bayer.com/kos/19050/PIDEntryLifecycleStatus"
    },
    nestedMetadata []
  },
...
]
```

If the property `"http://www.w3.org/ns/shacl#nodeKind"` has the value `"http://www.w3.org/ns/shacl#IRI"`, it points to another yet existing instance of an entity by it´s `id`. To get an overview which instances of en entity yet exists, the `Entity` API could be used in combination with the `http://www.w3.org/2000/01/rdf-schema#range` value as type.

There are some exceptions where special interfaces are build for, e.g. *ConsumerGroup*, *Keywords*, *ExtendedUriTemplate*, *PidUriTemplate*.

## API: Entity

This API has two interfaces which gives one the ability to query all instances of a given entity type but also the data of a specific instance by it´s id.

### The `entityList` interface

This interfaces delivers all instances of a given entity type. For this interface a limitation of the number of return values, but also an offset can be used.
Here the `https://pid.bayer.com/kos/19050/PIDEntryLifecycleStatus` entity from above example is used:

```shell
GET "hhttps://localhost:51770/api/v3/entityList?Limit=10&Offset=0&Type=https%3A%2F%2Fpid.bayer.com%2Fkos%2F19050%2FPIDEntryLifecycleStatus"
```

The response is a list of the instances including their data:

```json
[
  {
    "name": "Draft",
    "id": "https://pid.bayer.com/kos/19050/draft",
    "properties": {
      "http://www.w3.org/1999/02/22-rdf-syntax-ns#type": [
        "https://pid.bayer.com/kos/19050/PIDEntryLifecycleStatus"
      ],
      "http://pid.bayer.com/kos/19014/editorialNote": [
        "The status of the pid entry is draft."
      ],
      "http://www.w3.org/2000/01/rdf-schema#label": [
        "Draft"
      ]
    }
  },
  {
    "name": "Historic",
    "id": "https://pid.bayer.com/kos/19050/historic",
    "properties": {
      "http://www.w3.org/1999/02/22-rdf-syntax-ns#type": [
        "https://pid.bayer.com/kos/19050/PIDEntryLifecycleStatus"
      ],
      "http://www.w3.org/2000/01/rdf-schema#label": [
        "Historic"
      ]
    }
  },
  {
    "name": "Marked for deletion",
    "id": "https://pid.bayer.com/kos/19050/markedForDeletion",
    "properties": {
      "http://www.w3.org/1999/02/22-rdf-syntax-ns#type": [
        "https://pid.bayer.com/kos/19050/PIDEntryLifecycleStatus"
      ],
      "http://www.w3.org/2000/01/rdf-schema#label": [
        "Marked for deletion"
      ]
    }
  },
  {
    "name": "Published",
    "id": "https://pid.bayer.com/kos/19050/published",
    "properties": {
      "http://www.w3.org/1999/02/22-rdf-syntax-ns#type": [
        "https://pid.bayer.com/kos/19050/PIDEntryLifecycleStatus"
      ],
      "http://pid.bayer.com/kos/19014/editorialNote": [
        "The status of the pid entry is published."
      ],
      "http://www.w3.org/2000/01/rdf-schema#label": [
        "Published"
      ]
    }
  }
]
```

### The `entity` interface

This interfaces delivers the data of a specific instance of an entity by it´s `id`. Here the id of  `"https://pid.bayer.com/kos/19050/published"` from above example is used:

```shell
GET "https://localhost:51770/api/v3/entity?id=https%3A%2F%2Fpid.bayer.com%2Fkos%2F19050%2Fpublished"
```

The response contains the specific instance and it´s data:

```json
{
  "name": "Published",
  "id": "https://pid.bayer.com/kos/19050/published",
  "properties": {
    "http://www.w3.org/1999/02/22-rdf-syntax-ns#type": [
      "https://pid.bayer.com/kos/19050/PIDEntryLifecycleStatus"
    ],
    "http://www.w3.org/2000/01/rdf-schema#label": [
      "Published"
    ],
    "http://pid.bayer.com/kos/19014/editorialNote": [
      "The status of the COLID entry is published."
    ]
  }
}
```
