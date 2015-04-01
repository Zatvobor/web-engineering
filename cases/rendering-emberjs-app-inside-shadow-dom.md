_COMPLETELY OUTDATED_

## Rendering Ember.js application as a Web Component inside Shadow DOM

Everyone who does web engineering hard has already heard about HTML5 rocks called Shadow DOM. There is a [best place](http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom) for getting more instead. I wrote this because it took me some time to figure out how to render Ember.js app like a pure Web Component in context of [ember-insights.js](http://ember-insights.github.io) test case application.

This is a tiny explanation of getting work done which is based on distributed nodes approach.

### Don't waste your time for trying to render an app inside shadow-root

Web Component ships with its own `template`. Think about it like a plain `layout`. Web Component's JavaScript definitely can't modify parts inside the `layout`. This is a core feature addresses the DOM tree encapsulation problem.

### Distributed Nodes

In the beginning, let's get more about Distributed Nodes.

The Distributed nodes are elements that render at an insertion point (a `<content>` element). The `<content>` element allows you to select nodes  and render them at predefined locations in your Shadow DOM `layout`.

The Distributed nodes retain styles from the main document, even when they render at an insertion point and that is a problem.

#### The answer is the CSS `::content`

The `::content` pseudo element is a way to target Distributed nodes.

## The solution

```html
<!-- index.html -->
<link rel="import" href="example-component/initializer.html">
<link rel="import" href="example-component/app.html">
<div id="example-component-host"></div>
<script>
  var host      = document.querySelector('#example-component-host');
  var link      = document.querySelector('link[href="example-component/initializer.html"]');
  var template  = link.import.querySelector('template#application-layout');
  var clone     = document.importNode(template.content, true);
  host.appendChild(clone);
  template       = link.import.querySelector('template#shadow-layout');
  clone          = document.importNode(template.content, true);
  host.createShadowRoot().appendChild(clone);
</script>
```

```html
<!-- example-component/initializer.html -->
<template id="application-layout">
  <div id="example-component-container"></div>
  <script>
    var app_link      = document.querySelector('link[href="example-component/app.html"]');
    var app_template  = app_link.import.querySelector('template');
    var app_clone     = document.importNode(app_template.content, true);
    host.appendChild(app_clone);
  </script>
</template>

<template id="shadow-layout">
  <style>
    ::content #example-component-container {
      padding: 10px;
      margin: 10px;
    }
  </style>
  <div id="example-component-wrapper">
     <h4>application</h4>
     <content select="#example-component-container"></content>
   </div>
</template>
```

```html
<!-- example-component/app.html -->
<template>
  <script type="text/x-handlebars">
    {{outlet}}
  </script>
  <script>
    var App = Ember.Application.create({rootElement: '#example-component-container'});
  </script>
</template>

<script src="bower_components/jquery/dist/jquery.min.js"></script>
<script src="bower_components/handlebars/handlebars.min.js"></script>
<script src="bower_components/ember/ember.min.js"></script>
```

Finally, the main solution goal here is to be able to load it through HTML5 Imports. This is an example mostly addressed to be simple and clear about described approach.
