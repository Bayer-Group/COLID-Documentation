# Linking

When creating a COLID entry, different resources can refer to other resources. This creates a relationship between two entries so that both resources are linked to each other. Since these are edges between two objects, these edges can have different meanings. This document does not describe the edges and their meaning, but some of their edges are used in different examples. 

Two different concepts were developed and discussed during the implementation of the links. Each resource has a unique id in the triplestore and is used in COLID as an internal unique identifier. The PID URI is a property of a resource and is also unique. 

To implement a link, an edge between the two id of two resources must be realized and stored in the database. 
Instead of using the id (internal identifier) of a resource, it was agreed to use the PID URI for the storage, since this is known to every user, used in the UI and provided by the API. The internal identifier is not provided to the user by the API. 

## Data View

The following example shows a resource that should be saved. It has an example link that includes another ontology. From the metadata it can be seen that the edge has the value "https://pid.bayer.com/kos/19050/includesOntology". The associated value is the PID URI of the resource to be linked.

Request: 

``` json
    {
       "properties":[
          {
             "key":"https://pid.bayer.com/kos/19050/hasLabel",
             "value":"Demo Label"
          },
          {
             "key":"https://pid.bayer.com/kos/19050/includesOntology",
             "value":"https://dev-pid.bayer.com/8daf63f8-6e32-490b-9a65-7b4df89fffc3"
          },
          {
             "key":"http://www.w3.org/1999/02/22-rdf-syntax-ns#type",
             "value":"http://pid.bayer.com/kos/19014/Ontology"
          }
       ]
    }
```

The response contains more information than specified in the request. This has to do with the data transfer objects, as well as with some modifications that the data model requires especially for the frontend. Therefore, the PID URI is not displayed as the value as in the request, but the corresponding resource. The PID URI can then be found as the property of the resource. If you want to change the response and save it again, the response must be changed to the format of the request. Here you have to take care that all information of the linked resource has to be deleted and the PID URI has to be used again. 

``` json
    {
       [...]
       "id":"https://pid.bayer.com/kos/19050#6d279ab4-1fe8-43ae-8119-582d79121867",
       "properties":[
          {
             "key":"http://www.w3.org/1999/02/22-rdf-syntax-ns#type",
             "value":"http://pid.bayer.com/kos/19014/Ontology"
          },
          {
             "key":"http://pid.bayer.com/kos/19014/hasPID",
             "value":[
                {
                   "id":"https://dev-pid.bayer.com/a01bf7a8-ac7d-4ecb-a855-f22577ee6caf/",
                   "properties":[
                      {
                         "key":"http://www.w3.org/1999/02/22-rdf-syntax-ns#type",
                         "value":"https://pid.bayer.com/kos/19050/Identifier"
                      },
                      {
                         "key":"https://pid.bayer.com/kos/19050/hasUriTemplate",
                         "value":"https://pid.bayer.com/kos/19050#13cd004a-a410-4af5-a8fc-eecf9436b58b"
                      }
                   ]
                }
             ]
          },
          {
             "key":"https://pid.bayer.com/kos/19050/hasLabel",
             "value":"PID Ontology 1.0 (aka PID Metadata Ontology)"
          },
          {
             "key":"https://pid.bayer.com/kos/19050/includesOntology",
             "value":[
                {
                   "id":"https://pid.bayer.com/kos/19050#a67cf07b-2bed-43b3-9f02-98c5214d25f0",
                   "properties":[
                      {
                         "key":"http://www.w3.org/1999/02/22-rdf-syntax-ns#type",
                         "value":"http://pid.bayer.com/kos/19014/Ontology"
                      },
                      {
                         "key":"https://pid.bayer.com/kos/19050#hasConsumerGroup",
                         "value":"https://pid.bayer.com/kos/19050#2b3f0380-dd22-4666-a28b-7f1eeb82a5ff"
                      },
                      {
                         "key":"http://pid.bayer.com/kos/19014/hasPID",
                         "value":[
                            {
                               "id":"https://dev-pid.bayer.com/8daf63f8-6e32-490b-9a65-7b4df89fffc3",
                               "properties":[
                                  {
                                     "key":"http://www.w3.org/1999/02/22-rdf-syntax-ns#type",
                                     "value":"https://pid.bayer.com/kos/19050/Identifier"
                                  }
                               ]
                            }
                         ]
                      }
                   ]
                }
             ]
          }
       ]
    }
```

## Component View

As already described in another chapter, the components for display and editing are described by metadata and structured accordingly. Linked resources are also described by a metadata field in the Ontology and are handled separately and separately from other properties in the frontend and backend.

### Create und Edit

The **ResourceFormComponent** is used to create and edit a resource. This is implemented in the components **ResourceDisplayComponent** and **ResourceEditComponent**, depending on the operation. The ResourceFromComponent in turn implements the **FormComponents**. This component implements various **FormInputs**. These are selected and displayed according to data type and property identifier. This allows you to react to different data types and properties. These inputs have been extended within the scope of this request so that a selection for the linked resources is possible. 

## Backend

Since the Api user, who can be a different Api as well as the actual UI within the scope of using the application, defines the linking of the resource in the value of the property, more precisely in the request, by the PID URI. In addition to various validations, the backend replaces the PID URI with the internal identifier of the resource. Any process takes place in the ResourcePreProcessServices of the package PID.RegistrationService.Services. Since the response must not contain the PID URI but the information of a resource, the value is replaced after the CRUD operation. Thus the response does not contain a Key Value Pair from Property Key and PID URI but from Property Key and Resource. The component ResourceService in the package COLID.RegistrationService.Services contains the replacement of the value. 

## User Interface

For the implementation of the linked resources, extensive changes were made in the FrontEnd. 

### FormComponent

- Two functions have been implemented in the FormComponent, which once take care of checking the properties in order to check whether it is a link type. Furthermore, an event handler has been included that ensures that if a new resource is added in the FormItemCreateLinkingComponent, it is also added as a value to the current resource and thus also to Form. 
- The FormItemCreateLinkingComponent is displayed in the FormComponent if a property is a link type. 
 
### FormItemCreateLinkingComponent

- This component is responsible for adding the linked resource. If a property has the group "Link Types", this property will be replaced and added by this component. This component is responsible for providing and selecting the appropriate types and displaying the possible linked resources. Here the FormItemInputLinkingDialogComponent is used. 

If the component is closed, an event is executed that is passed on to the FormComponent: 

``` javascript
dialogRef.afterClosed().subscribe(result =&gt; {
  if (result) {
    console.log('Emit linkingEvent', result)
    this.linkingEvent.emit({ metadata: this.selectedLinkingType, resource: result});
  }
  this.hideLinkingTypes();
});
```

### FormItemInputLinkingDialogComponent

- This component consists of two additional components so that the SidebarFilterComponent and SidebarContentComponent are implemented and used. Please refer to the corresponding documentation. If a resource is selected in this dialog, it is passed as value to the FormItemCreateLinkingComponent when the dialog is closed and passed to the corresponding event. 

If the value was added to the resource and the corresponding form, the resource is extended by the corresponding link type. The component FormItemInputLinkingComponent is used for this. It is only used to display the linked resource with dedicated information and a function for deleting a link. This is also done using an event that removes the link after a confirmation dialog. 

``` javascript 
confirmAndRemoveLinking(linkEntity, index) {
    const dialogRef = this.dialog.open(DeleteItemDialogComponent, {
      data: {
        header: 'Deleting linking',
        body: `Are you sure you want to delete this linking: <br><br> <strong>PidUri:</strong>  ${linkEntity.pidUri}`
      },
      width: 'auto',
      disableClose: true
    });

    dialogRef.afterClosed().subscribe(result =&gt; {
      if (result) {
        this.internalValue.splice(index, 1); 
      }
    });
  }
```

### Backend

A new function called UpdateLinkingValue has been added to the ResourcePreprocessService class in the backend. This function is responsible for replacing the PID URI with the id of the resource. As already described in the previous chapter, the PID URI is used for the request. Within the function the PID URI fetches the main resource and replaces it with its id / identifier. This means if a published resource is available, it will be used, alternatively the draft resource. The system also checks whether the linked resource does not match the current resource and whether the data type of the linked resource corresponds to the data type prescribed by the metadata.  

A mapping is created between the PID URI and the replaced id to have the corresponding PID URI available in case of a Revert and not to load it again from the database. 

The mapping is used in the RevertResource function. Before the response is created with the current resource, this resource is enriched with the information of the linked resources so that the corresponding information is available to the user in the response. This is done in the ResourceService class after each Create/Edit. 

Note: Here you should check whether it is a high-performance solution in the future. Possibly such information should be loaded asynchronously by the user of the Api.

### Configuration Details

All properties which are link types and have the group "http://pid.bayer.com/kos/19050/LinkTypes" should not be considered in the filter of Data Marketplace, so that a ```facetExclude = true``` is implemented in the MetadataRepository.

``` c#
if (shaclValue == Constants.Resource.Group.LinkTypes)
{
  facetExclude = true;
}
```