Htmlizer
========

Generate HTML with templates that are valid HTML.

`npm install htmlizer`

### Why?

Most templating languages doesn't ensure that the templates are valid HTML. Templates needs to be parsable for build tools like assetgraph-builder to able to 1. find assets (like images) for optimization 2. Translate text with their [data-i18n](https://github.com/assetgraph/assetgraph-builder#html-i18n-syntax) syntax.

For example consider this Mustache template: `<div {{attributes}}></div>`.
This looks sane, but is unfortunately not parsable by most HTML parsers.

Here is another example: `<div style="{{style}}"></div>`. Even though this is parsable, the text inside the style attribute is not valid CSS syntax and some parsers or server-side DOM libraries (used within build tools) could throw errors.

### How?

The solution here is to use template syntax similar to KnockoutJS (in fact supports a subset of Knockout bindings).

Hence other use cases include, partially rendering a KO web page/app on the server-side for SEO purposes. 

## What's new in v2

### Performance improvement

With v2, to speed up performance on nodejs, jsdom as a dependency has been removed and htmlizer instead compiles templates into JS functions to generate the output. v2 only supports server-side rendering (unlike v1).

The following are some of the implications due to this change:

1. toDocumentFragment() method has been removed.
2. $element KO binding context cannot be used anymore.
3. template binding interface has changed.

Usage
-----

Render template as HTML string:
```
(new Htmlizer('<template string>')).toString(dataObject);
```

Template syntax
-----
Syntax is similar to KnockoutJS (in fact supports a subset of Knockout bindings).

#### *text* binding:

```
Template: <span data-bind="text: mytext"></span>

Data: {mytext: 'test'}

Output: <span>test</span>
```

#### *attr* binding:

```
Template: <span data-bind="text: mytext, attr: {class: cls}"></span>

Data: {mytext: 'test', cls: 'btn btn-default'}

Output: <span class="btn btn-default">test</span>
```

#### *if* binding:
```
Template:
<div data-bind="if: show">
  This message won't be shown.
</div>

Data: {show: false}

Output: <div></div>
```

#### Containerless *if* binding:
```
Template:
<div>
  <!-- ko if: count -->
    <div id="results"></div>
  <!-- /ko -->
  <!-- hz if: !count -->
    No results to display.
  <!-- /hz -->
</div>

Data: {count: 0}

Output: <div>No results to display.</div>
```

Note: You can use either "ko if:" or "hz if:" to begin an *if* statement. And you may either use "/ko" or "/hz" to end an *if* statement.

#### Containerless *text* binding:
```
Template:
<div>
  <!-- ko text: msg --><!-- /ko -->
</div>

Data: {msg: 'Hello'}

Output: <div>Hello</div>
```

#### *foreach* binding:
```
Template:
<div data-bind="foreach: items">
    <div data-bind="text: $data"></div>
</div>

Data:
{
  items: ['item 1', 'item 2', 'item 3']
}

Output:
<div>
  <div>item 1</div>
  <div>item 2</div>
  <div>item 3</div>
</div>
```

#### Containerless *foreach* binding:
```
Template:
<div>
  <!-- ko foreach: items -->
    <div data-bind="text: name"></div>
  <!-- /ko -->
</div>

Data:
{
    items: [{name: 'item 1'}, {name: 'item 2'}, {name: 'item 3'}]
}

Output:
<div>
  <div>item 1</div>
  <div>item 2</div>
  <div>item 3</div>
</div>
```

#### *html* binding:
```
Template:
<div data-bind="html: message"></div>

Data: {message: '<b>This</b> is a <b>serious message</b>'}

Output: <div><b>This</b> is a <b>serious message</b></div>
```

#### *css* binding:
```
Template:
<div data-bind="css: {warning: isWarning}"></div>

Data: {isWarning: true}

Output: <div class="warning"></div>
```

#### *style* binding:
```
Template:
<div data-bind="style: {fontWeight: bold ? 'bold' : 'normal'}"></div>

Data: {bold: false}

Output: <div style="font-weight: normal;"></div>
```

#### *with* binding:
```
Template:
<div data-bind="with: obj">
    <span data-bind="text: val"></span>
</div>

Data: {obj: {val: 10}}

Output:
<div>
    <span>10</span>
</div>
```

#### Containerless *with* binding:
```
Template:
<!-- ko with: obj -->
    <span data-bind="text: val"></span>
<!-- /ko -->

Data: {obj: {val: 10}}

Output: <span>10</span>
```

#### *template* binding:
Works mostly like KO 3.0 - [documentation](http://knockoutjs.com/documentation/template-binding.html).
Supports the following properties: name, data, if, foreach and as.

```
Template:
<div data-bind="template: { name: 'person-template', data: buyer }"></div>
<div data-bind="template: { name: 'person-template', foreach: [buyer] }"></div>

subtemplate.html:
<h3 data-bind="text: name"></h3>
<p>Credits: <span data-bind="text: credits"></span></p>

Data:
{
    buyer: {
        name: 'Franklin',
        credits: 250
    }
}

Output:
<div>
    <h3><Franklin/h3>
    <p>Credits: <span>250</span></p>
</div>
<div>
    <h3><Franklin/h3>
    <p>Credits: <span>250</span></p>
</div>
```

To make template work one must add the sub-templates to config as follows.

```
  var subtpl = require('fs').readFileSync('subtemplate.html').toString(), //load the file with the sub-templates.
      cfg = {
        templates: {
          "person-template": subtpl
        }
      },
      output = (new Htmlizer('<template string>', cfg).toString(dataObject);
```

#### Binding Contexts

Supports all but $element binding contexts documented for KO 3.0 [here](http://knockoutjs.com/documentation/binding-context.html).

```
Template:
<div data-bind="foreach: {data: items, as: 'obj'}">
    <!-- ko foreach: subItems -->
        <span data-bind="text: obj.name"></span>
        <span data-bind="text: $parent.name"></span>
        <span data-bind="text: $index"></span>
        <span data-bind="text: $data.name"></span>
        <span data-bind="text: name"></span>
        <span data-bind="text: $root === $parents[$parents.length - 1]"></span>
    <!-- /ko -->
</div>

Data:
{
    items: [{
        name: 'item1',
        subItems: [{
            name: 'subitem1'
        }]
    }]
}

Output:
<div>

        <span>item1</span>
        <span>item1</span>
        <span>0</span>
        <span>subitem1</span>
        <span>subitem1</span>
        <span>true</span>

</div>
```

Avoiding conflict with KO
-----
Use case: If you intend to do a partial render with Htmlizer on the server side and the rest with KO on the client side, then you will need to seperate concerns (i.e. avoid conflicts with KO).

To avoid conflict with KnockoutJS, set noConflict config to true:
```
var template = new Htmlizer('<template string>', {noConflict: true});
```
By default noConflict is assumed false. With noConflict = true, there are two main differences:

- Bindings must be placed within data-htmlizer attribute.
- Containerless statements with "ko" prefix will be ignored. Use "hz" prefix if you want Htmlizer to process it.

Keep KO bindings
-----
For certain use cases (like for SEO purpose without user agent sniffing), one would like to keep the bindings with the rendered output, so that KO on the client side can rewrite them. To keep KO bindings, set keepKOBindings config to true:
```
var template = new Htmlizer('<template string>', {keepKOBindings: true});
```
