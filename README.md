- Editor: [Daniel Beeke](mailto:mail@danielbeeke.nl)
- Last edit: May 5, 2023
- Status: In active development

A technology specification and RDF ontology for server driven user interfaces composed of RDF data and widgets (dash:Viewers). This specification allows for specific widgets (dash:Viewers) and layouts per frontend, these frontends may be websites but can also be native applications. This specification builds further on the [SHACL](https://www.w3.org/TR/shacl/) and [DASH](https://datashapes.org/forms.html) vocabularies.

## Introduction

Creating apps and websites can be tedious. Re-use of visual building blocks makes for a faster development phase. Re-using how things look also help developers to mostly focus on the business logic. 

SHACL is an excellent way for having portable data constraints. Similar to how a relational database has database constraints, a SHACL shape is a set of constraints on a specific type of RDF data.

DASH builds on top of this and enables us to have forms rendered from SHACL shapes. Another part that DASH has are dash:viewers. These dash:viewers can show data depending on configuration and the actual types of the data. It is resilient, the data may be entered incorrectly but it would still display according to their data types. 

The ViewMode specification builds further on these with:

- __SHACL ViewModes__, which is a SHACL shape catered to server driver Uis
- __Frontend ViewMode capabilities__ a specification for frontends to define what dash:Viewers they have available. It also allows a dash:Viewer to define settings that can be configured in the SHACL ViewMode.
- __ViewMode Layouts__ for use inside a sh:Group via vm:Layout. These layouts are living inside the frontend. They have some crude visual definition.
- __A SHACL ViewMode editor__ a suggestion how you could tie all these components together with a visual editor to make SHACL ViewModes.

## SHACL ViewModes

A SHACL ViewMode is a SHACL shape that only has SHACL properties that may be shown. Some SHACL properties might have a dash:viewer. There might be additional predicates inside a SHACL property, the predicates are defined by the SHACL shape of the specified dash:Viewer. 

### Required properties

A SHACL ViewMode MUST have the following properties inside the sh:NodeShape

- sh:targetClass

A SHACL ViewMode MAY have the following properties inside the sh:NodeShape

- sh:shapesGraph
- vm:interface
- vm:mode
- vm:layout

#### sh:interface

The sh:interface must be an URL to the frontend its __Frontend ViewMode capabilities__ document. If there are multiple documents possible it is allowed to have an ordered list. This helps with development so that your local development server has preference on the production server where still old code may be deployed. Intergrations always have to try the sh:interface values in order and use the first one that is responding with a HTTP 200.

### SHACL properties without an sh:path

A SHACL ViewMode may have SHACL properties without an sh:path. If no sh:path is given a vm:element MUST be set. The value must be something that inherits vm:Element. A vm:Element could be a favorite button, a share button or something similar, where the goal of the element is not represent some data part of a RDF document, but something about the document itself (for example: sharing it) or UI buttons that do something with the layout and have nothing to do with the data. Like increasing the font size.

### Example of a SHACL ViewMode

```Turtle
@prefix frontend: <https://example.com/> .
@prefix dash: <http://datashapes.org/dash#> .
@prefix shacl: <http://www.w3.org/ns/shacl#> .
@prefix vm: <https://viewmode.dev/ontology#> .

ex:PersonShapeViewMode
	a sh:NodeShape, vm:ViewMode ;
	sh:targetClass ex:Person ;
	sh:shapesGraph ex:PersonShape ;
	vm:mode frontend:ViewMode/card ;
	vm:layout frontend:TwoColumn ;
	vm:interface (
		<http://localhost:3000/viewmode.ttl>, 
		<https://example.com/viewmode.ttl>
	) ;

	sh:property [
		sh:path ex:name ;
		sh:maxCount 1 ;
		sh:datatype xsd:string ;
		sh:group frontend:TwoColumn/Left ;
	] ;

	sh:property [
		vm:element frontend:shareButtons ;
		sh:group frontend:TwoColumn/Top ;
	] ;

	sh:property [
		sh:path ex:ssn ;
		sh:maxCount 1 ;
		sh:datatype xsd:string ;
		sh:viewer frontend:ssn ;
		sh:group frontend:TwoColumn/Left ;
		rdfs:label "SSN" ;
		rdfs:description "The social security number of the person" ;
	] ;

	sh:property [
		sh:group frontend:TwoColumn/Right ;
		sh:path ex:about ;
		sh:datatype xsd:string ;
		rdfs:label "About" ;
	]

	sh:property [
		sh:group frontend:TwoColumn/Right ;
		sh:path ex:worksFor ;
		sh:class ex:Company ;
		sh:nodeKind sh:IRI ;
		sh:viewer [ vm:nestedViewMode ex:CompanyShapeViewMode ] ;
		rdfs:label "Company" ;
	]
.
```

## Frontend ViewMode capabilities

For server driven UI there is a relationship between the server and the frontend. The frontend defines which widgets (dash:Viewer and vm:Elements) are available and the server has configuration (a SHACL ViewMode) specific to a frontend in the datastore.

For a SHACL ViewMode editor to find out which widgets are available we have the concept: __Frontend ViewMode capabilities__. This is a document that is found at a URL. 

### Example of a __Frontend ViewMode capabilities__ document

```Turtle
@prefix frontend: <https://example.com/> .
@prefix dash: <http://datashapes.org/dash#> .
@prefix shacl: <http://www.w3.org/ns/shacl#> .
@prefix vm: <https://viewmode.dev/ontology#> .

frontend:ssn a dash:SingleViewer ;
  rdfs:label "SSN label" ;
  sh:node frontend:labelConfigurableShape, frontend:ssnConfigurableShape .

frontend:ssnConfigurableShape a sh:NodeShape ;
  sh:property [
		sh:path frontend:showDashes ;
		sh:maxCount 1 ;
		sh:datatype xsd:boolean ;
	] .

vm:nestedViewMode a dash:SingleViewer ;
  rdfs:label "Nested ViewMode" .
  sh:node frontend:nestedViewModeConfigurableShape .

frontend:nestedViewModeConfigurableShape a sh:NodeShape ;
  sh:property [
		sh:path vm:showLabel ;
		sh:maxCount 1 ;
		sh:datatype xsd:boolean ;
	] .

frontend:labelConfigurableShape a sh:NodeShape ;
  sh:property [
		sh:path rdfs:label ;
		sh:datatype sh:stringOrLangString ;
	] ;

  sh:property [
		sh:path vm:showLabel ;
		sh:maxCount 1 ;
		sh:datatype xsd:boolean ;
	] ;
  .

# And also layouts see in the next chapter.

```

## ViewMode Layouts

To tie everything together we also need some way of defining what kind of layouts the frontend supports.
Layouts enable use to group widgets in certain areas / groups of the layout.
The visual styles of these layouts need to be communicated in their most simple form. For this we use the CSS Grid syntax. 

These layouts can be included in a SHACL ViewMode renderer.

```Turtle

frontend:TwoColumn a vm:Layout ;
	vm:cssClass "two-columns" ;
	vm:gridStyles """
		display: grid;
		grid-template-columns: 1fr 300px;
		grid-template-rows: 1fr;
		grid-auto-rows: 1fr;
		gap: 0px 0px;
		grid-auto-flow: row;
		grid-template-areas: "left right";
	""" ;
	sh:group frontend:TwoColumn/Left, 
		frontend:TwoColumn/Right .

frontend:TwoColumn/Left
	a sh:PropertyGroup ;
	sh:order 0 ;
	vm:areaName "left" .

frontend:TwoColumn/Right
	a sh:PropertyGroup ;
	sh:order 1 ;
	vm:areaName "right" .

```

## A SHACL ViewMode editor

To get this system up and running we need an editor that enables us to create SHACL ViewModes and possibly SHACL shapes.

A SHACL ViewMode editor will support all dash:Viewers from the DASH specification if no vm:interface is given. When given a vm:interface it will only support the given dash:Viewers and vm:Elements.

If no SHACL shape is given, it will be possible to generate SHACL properties for arbitrary predicates. If a SHACL shape is given only the given predicates will be supported.