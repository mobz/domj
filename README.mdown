DOM transport via JSON (DOMj)
========================================================================================

This proposes a standard for describing the Document Object Model using Javascript Objects. Typically the DOM is described in HTML, however in a javascript based application, HTML is sub-optimal with regard to generation and transformation, typically requiring a templating solution. In addition a Javascript Object based syntax can represent data which is impossible or less precise than with text, such as dates, functions and [Files](https://developer.mozilla.org/en/DOM/File).

DOMj can be generated and transformed in browsers, on the server and be transported over networks.

Example
-------

Basic Example

    { tag: "DIV", id: "header", cls: "page-header" }
    
produces equivalent DOM to the following html ```<DIV id="header" class="page-header"></DIV>```

Reference Implementation
--------

 * [Reference Implementation](https://github.com/mobz/joey/blob/master/src/joey.js)
 * [Try It! - live coding](http://mobz.github.com/joey/fiddle.html)

Conformance
------

* DOMj specifies a subset of [valid ecmaScript](http://www.ecma-international.org/publications/standards/Ecma-262.htm) similar to JSON, but extended to allow additional datatypes.

* A powerful subset of DOMj _can_ be encoded in [JSON](http://json.org/) and allows for serialisation, storage and transport. This provides similar functionality to HTML.

Syntax
------

* DOM elements are represented by a simple javascript object `{ }`
* DOM properties are represented by a name value pair in the object. For example;
	* `id: "foo"`
	* `href: "http://github.com/"`
	* `disabled: true`
* Any object which inherits from the [HTMLElement](http://www.w3.org/TR/DOM-Level-2-HTML/html.html#ID-882764350) can be created
* Any property on any object which has a setter can be represented
* Several names are reserved and provide additional features
	* `tag` accepts a string and sets the element tagName when the element is created ( required )
	* `children` accepts an array and represents the child nodes of the element
	* `text` accepts a string and creates a single child TextNode
	* `dataset` accepts an object of name/value pairs that sets custom data attributes of the element
	* `style` accepts an object of name/value pairs that sets style properties of the element
	* `on(name)` accepts an eventListener and which is bound to the `name` event. The event is bound to the bubbling phase

Note that `children`, `text` `textContent` and `innerHTML` all set the children of the node and only one can be defined per node.


Environment
-----------

The primary use case for DOMj is in browser use as an intermediate data type (like JSON), rather than using a text only (html) templating engine.

DOMj can be used on the server side, to create an object which would be serialised and delivered to a browser for rendering ( rather than sending HTML )

DOMj produces DOM directly without passing through text based markup, so can be used in ```HTML```, ```XHTML```, ```SVG``` and ```XML``` documents. It can also be used in mixed environments, for example, creating SVG elements inside an HTML document.

DOMj does not need distinguish between self-closing tags and tags with empty bodies, as there are no distinction in DOM.

DOMj does not need to consider the encoding type of the underlying document, however the encoding of the source is paramount.

Property Specifics
==================

The ```tag``` Property
----------------------

* Sets the tagName of the element.
* Is required for top level object, for children it is optional, the default value is 'DIV'.
* In XML documents this attribute is case sensitive
* Uppercase is recommended in non-case sensitive documents
* Must follow the rules for [XML Names](http://www.w3.org/TR/REC-xml/#NT-Name).
	* In addition, The colon character (```:```) is not allowed (reserved for future support of namespaces)

Usage:

    { tag: "DIV" }
    
DOM Interface:

    document.createElement( _tag_ );

Setting Properties
------------------

* Set properties on an element
* Property must exist for that element class and must have a setter
* Property value must be of the appropriate type the property, or convertible to the appropriate type

Usage:

    { tag: "INPUT", type: "date", id: "the_id", valueAsDate: Date.now(), diabled: true } 

DOM Interface

    el[ _property_ ] = _value_

The `textContent` property
--------------------

* can not be used with ```children``` or ```innerHTML```
* html escaping of text is not required and html entities are not allowed
* special characters should be specified using javascript text escapes and unicode code points eg. "\n" (newline) or "\u2122" (trademark symbol)
* Is not effected by the encoding format of the document
* is typically aliased to the 'text' property (as below)

Usage:

    { tag: "B", text: "Some Bold Text" }

Dom Interface:

    el.textContent( _text_ );


The ```children``` property
-------------------

* defines children elements of a parent element
* Optional
* Accepts an array of objects
* objects may be
	* null - nothing is added - this is a noop
	* string - adds a TextNode
	* object - this is processed as a DOMj definition
	
Usage:

    { tag: "UL", children: [
        { tag: "LI", text: "Orange" },
        { tag: "LI", text: "Lemon" },
        { tag: "LI", text: "Lime" }
    ] }

    { tag: "P", children: [
        "Hey, check out this ",
        { tag: "A", href: "info.html", text: "link" },
        " for more information.",
        { tag: "BR" }
    ] }

    { tag: "TABLE", children: [
        { tag: "TBODY", children: [
            { tag: "TR", children: [
                { tag: "TD", text: "table cell!" }
            ] }
        ] }
    ] }

DOM Interface:

    for( i = 0; i < _children_.length; i++ ) {
        _childNode_ = _create_child_node_( _children_[i] );
        if( _childNode_ != null ) {
            el.appendChild( _childNode_ );
        }
    }

The ```on```* Property - binding eventListeners
-----------------------

* adds an event listener to the DOM element
* matches any property which starts with `on`
* derives an event name from the none 'on' part of the name
* supports both functions and handleEvent style objects
* bind the event in the bubbling phase.
* event names are case insensitive

Usage:

    { tag: "DIV", onclick: function( ev ) { alert( ev.type ); } }
    { tag: "DIV", onmouseover: this.mouseOverHandler }
    { tag: "DIV", ontouchstart: {
        touched: false,
        handleEvent: function() {
            this.touched = true;
        }
    } }

DOM Interface:

    _eventName_ = _property_.substr(2);
    el.addEventListener( _eventName_, _value_, false );

The ```style``` Mapped Property
-------------------------

* Sets css styles on the DOM element
* accepts a map of css property names, value pairs
* property names should be camel case
* the ';' separator is not allowed

Usage:

    { tag: "DIV", style: { color: "#ffeeaa", fontSize: "3em" } }
    { tag: "DIV", style: { zIndex: 3 } }
    { tag: "DIV", style: { background: "transparent url('foo.png') no-repeate top left" } }

DOM Interface:

    for( _cssProperty_ in _value_ ) {
        el.style.[ _cssProperty_ ] = _value_[ _cssProperty_ ];
    }


The ```dataset``` Mapped Property
-------------------------

* Sets custom data attributes on the DOM element
* accepts a map of data names, value pairs
* custom data attribute names should be camel case
* attribute values should be strings

Usage:

    { tag: "DIV", dataset: { "foo": "bar",  } }
    { tag: "DIV", dataset: { "someDataAttr": "attr value", "someOtherAttr": "different value" } }

DOM Interface:

    for( _dataAttributeName_ in _value_ ) {
        el.dataset[ _dataAttributeName_ ] = _value_[ _dataAttributeName_ ];
    }

Property Aliases
================

To improve readability and conciseness of DOMj, aliases to common properties are defined as below

* ```tag``` -> ```tagName```
* ```text``` -> ```textContent```
* ```cls``` -> ```className```






