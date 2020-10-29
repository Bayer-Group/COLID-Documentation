# UI generation with SHACLs

One of the special features of the Editor Client is that the display and editing dialogs are generated entirely from the metadata contained in the graph database and provided by the Registration Service. If a new COLID entry is created in the COLID Editor, the Registration Service loads the appropriate metadata for the selected resource type (e.g. Generic Dataset). Shacls are attached to this metadata, with which the COLID Editor can make various decisions:
- Is the field mandatory or not?
- Should only numbers and not letters be allowed?
- Is the field a date and should it therefore be possible to select from a date dialog?
- Is the field a single- or multiselect field and must correspondingly predefined values be loaded from the database?
- Is the field of type HTML and should a complete WYSIWYG HTML editor be displayed?

For these decisions own fieldTypes were created in the SHACL graph, which are loaded as additional subproperties with displays of the form from the Registration Service.

## Available form item types

There are many different types available, which can be set through the datatype field of the SHACLs.
By combining data types and other SHACL properties, the types of fields for the UI can be identified. This procedure is only of limited use. Taxonomies cannot be classified from controlled vocabulary using the SHACLs until the data is available.
Field types have been created in the metadata that specify exactly which types are meant. Similar to the official SHACL definition according to [w3](https://www.w3.org/ns/shacl), a namespace has been defined for COLID specific SHACLs:
* https://pid.bayer.com/ns/shacl

For field types is definied as shacl property:
* https://pid.bayer.com/ns/shacl/fieldType

### Distinction between field types as instances

* https://pid.bayer.com/ns/shacl#string
    + The field type string provides the user with a input field of type string. All characters may be entered.
* https://pid.bayer.com/ns/shacl#naturalNumber
    + The field type natural number provides the user with a input field of type number. You can enter natural numbers without a decimal place.
* https://pid.bayer.com/ns/shacl#number
     + The field type number provides the user with a input field of type number. Next to the check in the ui that only numbers with decimal digits can be entered, a regex must be stored in the metadata. This is the only way to verify that correct data is stored via api. 

* https://pid.bayer.com/ns/shacl#html
    + The field type html provides the user with a rich text editor. With this the user has the possibility to edit text as html and display it in the ui. It is also called: "what you see is what you get editor". [WYSIWYG]

* https://pid.bayer.com/ns/shacl#boolean
    + The field type boolean provides the user with a true - false input field. The user can choose between two ratio buttons in the ui.

* https://pid.bayer.com/ns/shacl#list
    + The field type list provides the user with a input fielf of type select. In the SHACLs must be defined that it is the nodekind IRI and has a range. 
     Example: 
     ```
     ?resource rdfs:range <http://pid.bayer.com/kos/19014/DataStorageSystem>;
     <http://www.w3.org/ns/shacl#property>
                [ a       <http://www.w3.org/ns/shacl#PropertyShape> ;
                  <http://www.w3.org/ns/shacl#nodeKind>
                          <http://www.w3.org/ns/shacl#IRI> .
     ```

* https://pid.bayer.com/ns/shacl#extendableList
    + The field type list provides the user with a input fielf of type select. In the SHACLs must be defined that it is the nodekind IRI and has a range. This type extends the standard type list `https://pid.bayer.com/ns/shacl#list` by the function that additional instances can be created. This makes the predefined list extendable. To use this function you have to add another property to the class used as range `https://pid.bayer.com/ns/shacl/instanceGraph`. This property specifies the graph in the database to be used to store the new instances.

    ```
     <http://pid.bayer.com/kos/19014/DataStorageSystem>; 
             rdfs:label       "Data Storage System"@en ;
             rdf:type         owl:Class ;
             <https://pid.bayer.com/ns/shacl/instanceGraph>  <https://pid.bayer.com/instances/DataStorageSystem> .
     ```

* https://pid.bayer.com/ns/shacl#hierarchy
    + The field type hierarchy provides the user with a input field of type select. In the SHACLs must be defined that it is the nodekind IRI and has a range like `https://pid.bayer.com/ns/shacl#list`. This field type should only be used for taxonomies.

* https://pid.bayer.com/ns/shacl#identifier
    + The field type list provides the user with a special input field. The user can create a permanent unique identifier with this field. In addition to selecting templates for generating identifiers, the user can also enter an identifier himself. Because this is a very special field, changes must be made to the software to add new identifier fields. 

* https://pid.bayer.com/ns/shacl#entity
    + This field provides the user with the possibility to add a new entity as property of the current entity. The nodekind as iri and a range must also be specified here.  The following property must also be stored in the metadata: 
    ```
    ?resource <http://topbraid.org/tosh#editWidget> <http://topbraid.org/swa#NestedObjectEditor> .
    ```

This enables the software to query the metadata for the type of the new entity and to display a corresponding nested form in the ui.

* https://pid.bayer.com/ns/shacl#linkedEntity
    + This field gives the user the possibility to link an entity with the current entity. According to the Range the user is shown the possible entities to be linked. It is important that the following properties are stored in the metadata:

    ```
    <https://pid.bayer.com/kos/19050/Mapping-hasTargetOntology> a <http://www.w3.org/ns/shacl#PropertyShape> ;
	<http://www.w3.org/ns/shacl#group> <http://pid.bayer.com/kos/19050/LinkTypes> ;
	<http://www.w3.org/ns/shacl#nodeKind> <http://www.w3.org/ns/shacl#IRI> ;
	<http://www.w3.org/ns/shacl#path> <https://pid.bayer.com/kos/19050/hasTargetOntology> ;
    ```

## General rules

The items of the form are sorted by different ordering numbers with different rules:
* Group have an order, which start at 1 to infinity
* SHACL properties themself have an order, which start at 1 to infinity
* All SHACL properties are in a group
* First the groups are ordered by order number, inside of the groups the SHACLs are sorted by their order number (sorting is done in backend before sending the metadata triples to the webclient)
* If a group is not set or the group order number is 999 (for technical information), the form items appear at the bottom

## Group order number rules

1. Reserved group for the PreferedLabel of the resource (Shown as header size 3)
2. Reserved group for the Definition of the resource (Shown as lead size paragraph)	
3. Group for technical information
4. Group for PID URI
5. Group for Target URI
6. Group for Usage and Maintenance
7. Group for SecurityAccess
8. Group for Quality

## Additional rules
### ReadOnly

* If the group order is not set, the field is set to readonly
* If the group order is set to 999, the field is set to readonly

### Single- and Multiselect

If the field type is defined as `https://pid.bayer.com/ns/shacl#list`, `https://pid.bayer.com/ns/shacl#hierarchy` or `https://pid.bayer.com/ns/shacl#extendableList`. This is a selection field in the ui.  This gives the user the possibility to select a pre-selected number of options. If the SHACL for maxCount `http://www.w3.org/ns/shacl#maxCount` is larger than 1, the field is used as multiselect, otherwise single selection possible only. 

### Multi Value Input - string only

If the field type is defined as `https://pid.bayer.com/ns/shacl#string` and the SHACL for maxCount ``http://www.w3.org/ns/shacl#maxCount` is larger than 1, the user can add multiple values for this input field.

### Is Mandatory
If the SHACL for minCount ``http://www.w3.org/ns/shacl#minCount` is set to 1, a red star (*) after the name is shown as a mandatory field

### Number

Since different databases deal with numbers differently, it was decided that even numbers stored with the data type `http://www.w3.org/2001/XMLSchema#string`. 
To prevent the input of invalid characters, these fields are provided with a regex. Currently the regex `^[0-9]*$` is used. The user interface checks for this regex and displays a corresponding inout field of type number.

Although input fields of type `https://pid.bayer.com/ns/shacl#number` or `https://pid.bayer.com/ns/shacl#decimal` are also displayed for the following data types, it can lead to problems as described above:

`http://www.w3.org/2001/XMLSchema#float`
`http://www.w3.org/2001/XMLSchema#decimal`

 
## Examples
### Groups
```
<https://pid.bayer.com/kos/19050#TechnicalInformation>
        a       <http://www.w3.org/ns/shacl#PropertyGroup> ;
        <http://www.w3.org/2000/01/rdf-schema#label>
                "Technical Information"@en ;
        <http://topbraid.org/tosh#editGroupDescription>
                "Grouping all not editable technical Information"@en ;
        <http://topbraid.org/tosh#viewGroupDescription>
                "Grouping all not editable technical Information"@en ;
        <http://www.w3.org/ns/shacl#order>
                "3"^^<http://www.w3.org/2001/XMLSchema#decimal> .

<https://pid.bayer.com/kos/19050#UsageAndMaintenance>
        a       <http://www.w3.org/ns/shacl#PropertyGroup> ;
        <http://www.w3.org/2000/01/rdf-schema#label>
                "Usage and Maintenance"@en ;
        <http://topbraid.org/tosh#editGroupDescription>
                "Grouping all Information for Usage and Maintenance"@en ;
        <http://topbraid.org/tosh#viewGroupDescription>
                "Grouping all Information for Usage and Maintenance"@en ;
        <http://www.w3.org/ns/shacl#order>
                "6"^^<http://www.w3.org/2001/XMLSchema#decimal> .
```

### SHACLs
```
<https://pid.bayer.com/kos/19050#hasBusinessDomain>
        <http://www.w3.org/ns/shacl#property>
                [ a       <http://www.w3.org/ns/shacl#PropertyShape> ;
                  <http://www.w3.org/ns/shacl#datatype>
                          <http://www.w3.org/2001/XMLSchema#dateTime> ;
                  <http://www.w3.org/ns/shacl#group>
                          <https://pid.bayer.com/kos/19050#TechnicalInformation> ;
                  <http://www.w3.org/ns/shacl#maxCount>
                          1 ;
                  <http://www.w3.org/ns/shacl#minCount>
                          1 ;
                  <http://www.w3.org/ns/shacl#name>
                          "Last change date and time"@en ;
                  <http://www.w3.org/ns/shacl#nodeKind>
                          <http://www.w3.org/ns/shacl#Literal> ;
                  <http://www.w3.org/ns/shacl#order>
                          "2"^^<http://www.w3.org/2001/XMLSchema#decimal> ;
                  <http://www.w3.org/ns/shacl#path>
                          <https://pid.bayer.com/kos/19050#dateModified>
                ] ;
        <http://www.w3.org/ns/shacl#property>
                [ a       <http://www.w3.org/ns/shacl#PropertyShape> ;
                  <http://www.w3.org/ns/shacl#group>
                          <https://pid.bayer.com/kos/19050#UsageAndMaintenance> ;
                  <http://www.w3.org/ns/shacl#maxCount>
                          5 ;
                  <http://www.w3.org/ns/shacl#minCount>
                          1 ;
                  <http://www.w3.org/ns/shacl#name>
                          "Business Domain"@en ;
                  <http://www.w3.org/ns/shacl#nodeKind>
                          <http://www.w3.org/ns/shacl#IRI> ;
                  <http://www.w3.org/ns/shacl#order>
                          "3"^^<http://www.w3.org/2001/XMLSchema#decimal> ;
                  <http://www.w3.org/ns/shacl#path>
```
