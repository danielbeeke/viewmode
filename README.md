# Viewmode

A technology specification and RDF ontology (both in development) for having configurable views of RDF data on top of the [SHACL](https://www.w3.org/TR/shacl/) and [DASH](https://datashapes.org/forms.html) vocabularies for an application, regardless of any implementation stack, but mainly focused on websites.

## Introduction

Creating apps and websites can be tedious. Re-use of visual building blocks makes for a faster development phase. Re-using how things look would enable for applications that mostly focus on the business logic. 

SHACL is an excellent way to have portable data constraints. Similar to how a relation database has database constraints, a SHACL shape is a set of constraints on a specific type of RDF data.

DASH builds on top of this and enables us to have forms rendered from SHACL shapes. Another part that DASH has are dash:viewers. These dash:viewers can show data depending on configuration and the actual types of the data. It is resiliant, the data may be entered incorrectly but it would still display.

The ViewMode specification aims to bridge the gap from SHACL and DASH to a workable implementation that supports different frontends to gather the available dash:editors and dash:viewers from, have viewers without an sh:path.

Another part that needs bridging is a way of glueing it all together. This specification also will propose an editor for SHACL shapes to create SHACL ViewModes (a SHACL shape focused on the dash:viewers).

## General idea

You will have at least one SHACL shape for your form and render that with a SHACL form renderer. Another SHACL shape is aimed at displaying, will be made. To make these a visual administrative interface for creating and editing these shapes would be great.

Parts:

- SHACL form renderder (not part of this spec, an initiative is starting inside the rdfjs/public Gitter group)
- SHACL ViewMode editor
- SHACL ViewMode renderer

## What is a SHACL ViewMode?

A SHACL ViewMode is an ordinary SHACL shape but it does not have dash:editor predicates. It does have dash:viewer predicates. It is only focused on displaying the data versus creating and editing.

### Why would we need a seperate document / shape for viewing?

A SHACL ViewMode might not display all the SHACL properties that a form has. 

## What is a SHACL (ViewMode) editor?

A SHACL ViewMode editor is a visual user interface for an end-user to create a SHACL shape. This editor can be used both for creating standard SHACL shapes for the purpose of validation or forms and it can be used to create SHACL ViewModes.

This editor could be a visual editor where you can make nested layouts with sh:group predicates. And it can connect to a frontend and request all the available dash:editors and dash:viewers and their vm:configurables.

TBD

## What is a SHACL ViewMode renderer?

TBD