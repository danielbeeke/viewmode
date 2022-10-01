# Viewmode

A technology specification and RDF ontology for having configurable view modes on top of SHACL shapes for a frontend website.

## Introduction

Where do you have the configuration of display widgets that define how a piece of data looks? With a headless CMS this often shifts to the frontend. But what if you would like to have your frontend highly customizable with a UI?

## Components

### Frontend preparation

- Every value for a predicate in your SHACL shape can have a different display widget and possible configuration for the widget.
- Create widgets that are JavaScript objects, functions or classes.
- Add static properties defining configurables (booleans, enums and inputs).
- Export all your widgets configurables from one ES module. You can statically exports this as JSON or as a JavaScript object.
- Enable [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) for your homepage and the widgets ES module.

### Admin UI

- Use [SHACL shape](https://www.w3.org/TR/shacl/) for defining your data, make sure they are accesible by the admin UI. Basic Auth is possible.
- Use the UI CustomElement: 
```HTML
<viewmode-ui 
  shacl="https://backend.com/shapes/person.shacl.ttl"
  widgets="https://example.com/widgets.mjs"
/>
```

```JavaScript
const ui = document.querySelector('viewmode-ui')
ui.addEventListener('save', (turtle) => {
  // Save the viewmode turtle file to your backend.
})
```

### Frontend render

- When rendering the frontend make sure to load the viewmode turtle file and render accordingly.

```Turtle
PREFIX viewmode: <http://example.com/viewmodes/>
PREFIX widget: <http://example.com/widgets/>
PREFIX vm: <http://viewmode.danielbeeke.nl/ontology#>

viewmode:full
  vm:shape <https://my-backend.com/person.shape.ttl> ;
  vm:wrapper "<div class='person full'>" ;

  vm:group [
    a vm:Group ;
    vm:items [

      vm:item [
        a vm:Item ;
        vm:predicate schema:name ;
        vm:widget widgets:Label ;
        vm:wrapper "<div class='name'>"
        vm:config [
          widgets:Label/bold true ;
        ] ;
      ] ;

    ] ;
  ] ;


```

