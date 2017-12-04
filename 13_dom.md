{{meta {chap_num: 13, prev_link: 12_browser, next_link: 14_event, load_files: ["code/mountains.js", "code/chapter/13_dom.js"]}}}

# The Document Object Model

{{index drawing, parsing}}

When you open a web page in your browser, the browser
retrieves the page's ((HTML)) text and parses it, much like the way
our parser from [Chapter ?](language#parsing) parsed
programs. The browser builds up a model of the document's
((structure)) and then uses this model to draw the page on the screen.

{{index "live data structure"}}

This representation of the ((document))
is one of the toys that a JavaScript program has
available in its ((sandbox)). You can read from the model and also change it. It acts as a
_live_ data structure: when it is modified, the page on the screen is
updated to reflect the changes.

## Document structure

You can imagine an ((HTML)) document as a nested set of ((box))es.
Tags such as `<body>` and `</body>` enclose other ((tag))s, which in
turn contain other tags or ((text)). Here's the example document from
the [previous chapter](browser):

```{lang: "text/html", sandbox: "homepage"}
<!doctype html>
<html>
  <head>
    <title>My home page</title>
  </head>
  <body>
    <h1>My home page</h1>
    <p>Hello, I am Marijn and this is my home page.</p>
    <p>I also wrote a book! Read it
      <a href="http://eloquentjavascript.net">here</a>.</p>
  </body>
</html>
```

This page has the following structure:

{{figure {url: "img/html-boxes.svg", alt: "HTML document as nested boxes",width: "7cm"}}}

{{indexsee "Document Object Model", DOM}}

The data structure the browser uses to represent the document follows
this shape. For each box, there is an ((object)), which we can
interact with to find out things such as what HTML tag it represents and
which boxes and text it contains. This representation is called the
_Document Object Model_, or ((DOM)) for short.

{{index "documentElement property", "head property", "body property", "html (HTML tag)", "body (HTML tag)", "head (HTML tag)"}}

The global variable `document` gives us access to these
objects. Its `documentElement` property refers to the object
representing the `<html>` tag. It also provides the properties `head` and
`body`, which hold the objects for those elements.

## Trees

{{index [nesting, "of objects"]}}

Think back to the ((syntax tree))s from
[Chapter ?](language#parsing) for a moment. Their
structures are strikingly similar to the structure of a browser's
document. Each _((node))_ may refer to other nodes, _children_, which
in turn may have their own children. This shape is typical of nested
structures where elements can contain sub-elements that are similar to
themselves.

{{index "documentElement property"}}

We call a data structure a _((tree))_
when it has a branching structure, has no ((cycle))s (a node may not
contain itself, directly or indirectly), and has a single,
well-defined “((root))”. In the case of the ((DOM)),
`document.documentElement` serves as the root.

{{index sorting, "data structure", "syntax tree"}}

Trees come up a lot
in computer science. In addition to representing recursive structures such as
HTML documents or programs, they are often used to maintain
sorted ((set))s of data because elements can usually be found or
inserted more efficiently in a sorted tree than in a sorted flat
array.

{{index "leaf node", "Egg language"}}

A typical tree has different kinds of ((node))s. The syntax tree for
[the Egg language](language) had variables, values, and application
nodes. Application nodes always have children, whereas variables and
values are _leaves_, or nodes without children.

{{index "body property"}}

The same goes for the DOM. Nodes for regular
_((element))s_, which represent ((HTML)) tags, determine the structure
of the document. These can have ((child node))s. An example of such a
node is `document.body`. Some of these children can be ((leaf node))s,
such as pieces of ((text)) or ((comment))s (comments are written between
`<!--` and `-->` in HTML).

{{index "text node", "ELEMENT_NODE code", "COMMENT_NODE code", "TEXT_NODE code", "nodeType property"}}

Each DOM node object
has a `nodeType` property, which contains a numeric code that
identifies the type of node. Regular elements have the value 1, which
is also defined as the constant property `document.ELEMENT_NODE`. Text
nodes, representing a section of text in the document, have the value
3 (`document.TEXT_NODE`). Comments have the value 8
(`document.COMMENT_NODE`).

So another way to visualize our document ((tree)) is as follows:

{{figure {url: "img/html-tree.svg", alt: "HTML document as a tree",width: "8cm"}}}

The leaves are text nodes, and the arrows indicate parent-child
relationships between nodes.

{{id standard}}
## The standard

{{index "programming language", [interface, design]}}

Using cryptic numeric
codes to represent node types is not a very JavaScript-like thing to
do. Later in this chapter, we'll see that other parts of the
((DOM)) interface also feel cumbersome and alien. The reason for this
is that the DOM wasn't designed for just JavaScript. Rather, it tries
to define a language-neutral ((interface)) that can be used in other
systems as well—not just HTML but also ((XML)), which is a generic
((data format)) with an HTML-like syntax.

{{index consistency, integration}}

This is unfortunate. Standards are
often useful. But in this case, the advantage (cross-language
consistency) isn't all that compelling. Having an interface that is
properly integrated with the language you are using will save you more
time than having a familiar interface across languages.

{{index "array-like object", "NodeList type"}}

As an example of such poor
integration, consider the `childNodes` property that element nodes in
the DOM have. This property holds an array-like object, with a
`length` property and properties labeled by numbers to access the
child nodes. But it is an instance of the `NodeList` type, not a real
array, so it does not have methods such as `slice` and `forEach`.

{{index [interface, design], [DOM, construction], "side effect"}}

Then
there are issues that are simply poor design. For example, there is no
way to create a new node and immediately add children or attributes to
it. Instead, you have to first create it, then add the children one by
one, and finally set the attributes one by one, using side effects. Code that
interacts heavily with the DOM tends to get long, repetitive, and
ugly.

{{index library}}

But these flaws aren't fatal. Since JavaScript
allows us to create our own ((abstraction))s, it is easy to write some
((helper function))s that allow you to express the operations you are
performing in a clearer and shorter way. In fact, many libraries
intended for browser programming come with such tools.

## Moving through the tree

{{index pointer}}

DOM nodes contain a wealth of ((link))s to other nearby
nodes. The following diagram illustrates these:

{{figure {url: "img/html-links.svg", alt: "Links between DOM nodes",width: "6cm"}}}

{{index "child node", "parentNode property", "childNodes property"}}

Although the diagram shows only one link of each type,
every node has a `parentNode` property that points to its containing
node. Likewise, every element node (node type 1) has a `childNodes`
property that points to an ((array-like object)) holding its children.

{{index "firstChild property", "lastChild property", "previousSibling property", "nextSibling property"}}

In theory, you could move
anywhere in the tree using just these parent and child links. But
JavaScript also gives you access to a number of additional convenience
links. The `firstChild` and `lastChild` properties point to the first
and last child elements or have the value `null` for nodes without
children. Similarly, `previousSibling` and `nextSibling` point to
adjacent nodes, which are nodes with the same parent that appear immediately
before or after the node itself. For a first child, `previousSibling`
will be null, and for a last child, `nextSibling` will be null.

{{index "talksAbout function", recursion, [nesting, "of objects"]}}

When
dealing with a nested data structure like this one, recursive functions
are often useful. The following recursive function scans a document for ((text node))s
containing a given string and returns `true` when it has found one:

{{id talksAbout}}
```{sandbox: "homepage"}
function talksAbout(node, string) {
  if (node.nodeType == document.ELEMENT_NODE) {
    for (var i = 0; i < node.childNodes.length; i++) {
      if (talksAbout(node.childNodes[i], string))
        return true;
    }
    return false;
  } else if (node.nodeType == document.TEXT_NODE) {
    return node.nodeValue.indexOf(string) > -1;
  }
}

console.log(talksAbout(document.body, "book"));
// → true
```

{{index "nodeValue property"}}

The `nodeValue` property of a text node refers
to the string of text that it represents.

## Finding elements

{{index DOM, "body property", "hard-coding"}}

Navigating these
((link))s among parents, children, and siblings is often useful, as in
the previous function, which runs through the whole document. But if we
want to find a specific node in the document, reaching it by starting
at `document.body` and blindly following a hard-coded path of links is
a bad idea. Doing so bakes assumptions into our program about the
precise structure of the document—a structure we might want to change
later. Another complicating factor is that text nodes are created even
for the ((whitespace)) between nodes. The example document's body tag
does not have just three children (`<h1>` and two `<p>` elements) but
actually has seven: those three, plus the spaces before, after, and
between them.

{{index searching, "href attribute", "getElementsByTagName method"}}

So
if we want to get the `href` attribute of the link in that document,
we don't want to say something like “Get the second child of the sixth
child of the document body”. It'd be better if we could say “Get the
first link in the document”. And we can.

```{sandbox: "homepage"}
var link = document.body.getElementsByTagName("a")[0];
console.log(link.href);
```

{{index "child node"}}

All element nodes have a `getElementsByTagName`
method, which collects all elements with the given tag name that are
descendants (direct or indirect children) of the given node and
returns them as an array-like object.

{{index "id attribute", "getElementById method"}}

To find a specific
_single_ node, you can give it an `id` attribute and use
`document.getElementById` instead.

```{lang: "text/html"}
<p>My ostrich Gertrude:</p>
<p><img id="gertrude" src="img/ostrich.png"></p>

<script>
  var ostrich = document.getElementById("gertrude");
  console.log(ostrich.src);
</script>
```

{{index "getElementsByClassName method", "class attribute"}}

A third,
similar method is `getElementsByClassName`, which, like
`getElementsByTagName`, searches through the contents of an element
node and retrieves all elements that have the given string in their
`class` attribute.

## Changing the document

{{index "side effect", "removeChild method", "appendChild method", "insertBefore method", [DOM, construction]}}

Almost
everything about the ((DOM)) data structure can be changed. Element
nodes have a number of methods that can be used to change their
content. The `removeChild` method removes the given child node from
the document. To add a child, we can use `appendChild`, which puts it
at the end of the list of children, or `insertBefore`, which inserts
the node given as the first argument before the node given as the second
argument.

```{lang: "text/html"}
<p>One</p>
<p>Two</p>
<p>Three</p>

<script>
  var paragraphs = document.body.getElementsByTagName("p");
  document.body.insertBefore(paragraphs[2], paragraphs[0]);
</script>
```

A node can exist in the document in only one place. Thus, inserting
paragraph “Three” in front of paragraph “One” will first remove it
from the end of the document and then insert it at the front,
resulting in “Three/One/Two”. All operations that insert a node
somewhere will, as a ((side effect)), cause it to be removed from its
current position (if it has one).

{{index "insertBefore method", "replaceChild method"}}

The `replaceChild`
method is used to replace a child node with another one. It takes as
arguments two nodes: a new node and the node to be replaced. The
replaced node must be a child of the element the method is called on.
Note that both `replaceChild` and `insertBefore` expect the _new_ node
as their first argument.

## Creating nodes

{{index "alt attribute", "img (HTML tag)"}}

In the following example, we
want to write a script that replaces all ((image))s (`<img>` tags) in
the document with the text held in their `alt` attributes, which
specifies an alternative textual representation of the image.

{{index "createTextNode method"}}

This involves not only removing the images
but adding a new text node to replace them. For this, we use the
`document.createTextNode` method.

```{lang: "text/html"}
<p>The <img src="img/cat.png" alt="Cat"> in the
  <img src="img/hat.png" alt="Hat">.</p>

<p><button onclick="replaceImages()">Replace</button></p>

<script>
  function replaceImages() {
    var images = document.body.getElementsByTagName("img");
    for (var i = images.length - 1; i >= 0; i--) {
      var image = images[i];
      if (image.alt) {
        var text = document.createTextNode(image.alt);
        image.parentNode.replaceChild(text, image);
      }
    }
  }
</script>
```

{{index "text node"}}

Given a string, `createTextNode` gives us a type 3 DOM
node (a text node), which we can insert into the document to make it
show up on the screen.

{{index "live data structure", "getElementsByTagName method", "childNodes property"}}

The loop that goes over the images
starts at the end of the list of nodes. This is necessary because the
node list returned by a method like `getElementsByTagName` (or a
property like `childNodes`) is _live_. That is, it is updated as the
document changes. If we started from the front, removing the first
image would cause the list to lose its first element so that the
second time the loop repeats, where `i` is 1, it would stop because
the length of the collection is now also 1.

{{index "slice method"}}

If you want a _solid_ collection of nodes, as
opposed to a live one, you can convert the collection to a real array
by calling the array `slice` method on it.

```
var arrayish = {0: "one", 1: "two", length: 2};
var real = Array.prototype.slice.call(arrayish, 0);
real.forEach(function(elt) { console.log(elt); });
// → one
//   two
```

{{index "createElement method"}}

To create regular ((element)) nodes (type
1), you can use the `document.createElement` method. This method takes
a tag name and returns a new empty node of the given type.

{{index "Popper, Karl", [DOM, construction], "elt function"}}

{{id elt}}
The
following example defines a utility `elt`, which creates an element
node and treats the rest of its arguments as children to that node.
This function is then used to add a simple attribution to a quote.

```{lang: "text/html"}
<blockquote id="quote">
  No book can ever be finished. While working on it we learn
  just enough to find it immature the moment we turn away
  from it.
</blockquote>

<script>
  function elt(type) {
    var node = document.createElement(type);
    for (var i = 1; i < arguments.length; i++) {
      var child = arguments[i];
      if (typeof child == "string")
        child = document.createTextNode(child);
      node.appendChild(child);
    }
    return node;
  }

  document.getElementById("quote").appendChild(
    elt("footer", "—",
        elt("strong", "Karl Popper"),
        ", preface to the second editon of ",
        elt("em", "The Open Society and Its Enemies"),
        ", 1950"));
</script>
```

{{if book

This is what the resulting document looks like:

{{figure {url: "img/blockquote.png", alt: "A blockquote with attribution",width: "8cm"}}}

if}}

## Attributes

{{index "href attribute"}}

Some element ((attribute))s, such as `href` for
links, can be accessed through a ((property)) of the same name on the
element's ((DOM)) object. This is the case for a limited set of
commonly used standard attributes.

{{index "data attribute", "getAttribute method", "setAttribute method"}}

But HTML allows you to set any attribute you want on nodes.
This can be useful because it allows you to store extra information in a
document. If you make up your own attribute names, though, such
attributes will not be present as a property on the element's node.
Instead, you'll have to use the `getAttribute` and `setAttribute`
methods to work with them.

```{lang: "text/html"}
<p data-classified="secret">The launch code is 00000000.</p>
<p data-classified="unclassified">I have two feet.</p>

<script>
  var paras = document.body.getElementsByTagName("p");
  Array.prototype.forEach.call(paras, function(para) {
    if (para.getAttribute("data-classified") == "secret")
      para.parentNode.removeChild(para);
  });
</script>
```

I recommended prefixing the names of such made-up attributes with
`data-` to ensure they do not conflict with any other
attributes.

{{index "programming language", "syntax highlighting example"}}

As a simple
example, we'll write a “syntax highlighter” that looks for `<pre>`
tags (“preformatted”, used for code and similar plaintext) with a
`data-language` attribute and crudely tries to highlight the
((keyword))s for that language.

```{sandbox: "highlight", includeCode: true}
function highlightCode(node, keywords) {
  var text = node.textContent;
  node.textContent = ""; // Clear the node

  var match, pos = 0;
  while (match = keywords.exec(text)) {
    var before = text.slice(pos, match.index);
    node.appendChild(document.createTextNode(before));
    var strong = document.createElement("strong");
    strong.appendChild(document.createTextNode(match[0]));
    node.appendChild(strong);
    pos = keywords.lastIndex;
  }
  var after = text.slice(pos);
  node.appendChild(document.createTextNode(after));
}
```

{{index "pre (HTML tag)", "syntax highlighting example", "highlightCode function"}}

The function `highlightCode` takes a `<pre>` node and a
((regular expression)) (with the “global” option turned on) that
matches the keywords of the programming language that the element
contains.

{{index "strong (HTML tag)", clearing, "textContent property"}}

The
`textContent` property is used to get all the ((text)) in the node
and is then set to an empty string, which has the effect of emptying
the node. We loop over all matches of the keyword expression,
appending the text _between_ them as regular text nodes, and the text
matched (the keywords) as text nodes wrapped in `<strong>` (bold) elements.

{{index "data attribute", "getElementsByTagName method"}}

We can
automatically highlight all programs on the page by looping over all
the `<pre>` elements that have a `data-language` attribute and
calling `highlightCode` on each one with the correct regular
expression for the language.

```{sandbox: "highlight", includeCode: true}
var languages = {
  javascript: /\b(function|return|var)\b/g /* … etc */
};

function highlightAllCode() {
  var pres = document.body.getElementsByTagName("pre");
  for (var i = 0; i < pres.length; i++) {
    var pre = pres[i];
    var lang = pre.getAttribute("data-language");
    if (languages.hasOwnProperty(lang))
      highlightCode(pre, languages[lang]);
  }
}
```

{{index "syntax highlighting example"}}

Here is an example:

```{lang: "text/html", sandbox: "highlight"}
<p>Here it is, the identity function:</p>
<pre data-language="javascript">
function id(x) { return x; }
</pre>

<script>highlightAllCode();</script>
```

{{if book

This produces a page that looks like this:

{{figure {url: "img/highlighted.png", alt: "A highlighted piece of code",width: "4.8cm"}}}

if}}

{{index "getAttribute method", "setAttribute method", "className property", "class attribute"}}

There is one commonly used attribute,
`class`, which is a ((reserved word)) in the JavaScript language. For
historical reasons—some old JavaScript implementations could not
handle property names that matched keywords or reserved words—the
property used to access this attribute is called `className`. You can
also access it under its real name, `"class"`, by using the
`getAttribute` and `setAttribute` methods.

## Layout

{{index layout, "block element", "inline element", "p (HTML tag)", "h1 (HTML tag)", "a (HTML tag)", "strong (HTML tag)"}}

You
might have noticed that different types of elements are laid out
differently. Some, such as paragraphs (`<p>`) or headings (`<h1>`),
take up the whole width of the document and are rendered on separate
lines. These are called _block_ elements. Others, such as links
(`<a>`) or the `<strong>` element used in the previous example, are
rendered on the same line with their surrounding text. Such elements
are called _inline_ elements.

{{index drawing}}

For any given document, browsers are able to compute a
layout, which gives each element a size and position based on its
type and content. This layout is then used to actually draw the
document.

{{index "border (CSS)", "offsetWidth property", "offsetHeight property", "clientWidth property", "clientHeight property", dimensions}}

The size and position of an element can be
accessed from JavaScript. The `offsetWidth` and `offsetHeight`
properties give you the space the element takes up in _((pixel))s_. A
pixel is the basic unit of measurement in the browser and typically
corresponds to the smallest dot that your screen can display.
Similarly, `clientWidth` and `clientHeight` give you the size of the
space _inside_ the element, ignoring border width.

```{lang: "text/html"}
<p style="border: 3px solid red">
  I'm boxed in
</p>

<script>
  var para = document.body.getElementsByTagName("p")[0];
  console.log("clientHeight:", para.clientHeight);
  console.log("offsetHeight:", para.offsetHeight);
</script>
```

{{if book

Giving a paragraph a border causes a rectangle to be drawn around it.

{{figure {url: "img/boxed-in.png", alt: "A paragraph with a border",width: "8cm"}}}

if}}

{{index "getBoundingClientRect method", position, "pageXOffset property", "pageYOffset property"}}

{{id boundingRect}}
The most effective way to find
the precise position of an element on the screen is the
`getBoundingClientRect` method. It returns an object with `top`,
`bottom`, `left`, and `right` properties, indicating the pixel
positions of the sides of the element relative to the top left of the
screen. If you want them relative to the whole document, you must
add the current scroll position, found under the global `pageXOffset`
and `pageYOffset` variables.

{{index "offsetHeight property", "getBoundingClientRect method", drawing, laziness, performance, efficiency}}

Laying
out a document can be quite a lot of work. In the interest of speed,
browser engines do not immediately re-layout a document every time it
is changed but rather wait as long as they can. When a JavaScript
program that changed the document finishes running, the browser will
have to compute a new layout in order to display the changed document
on the screen. When a program _asks_ for the position or size of
something by reading properties such as `offsetHeight` or calling
`getBoundingClientRect`, providing correct information also requires
computing a ((layout)).

{{index "side effect", optimization, benchmark}}

A program that
repeatedly alternates between reading DOM layout information and
changing the DOM forces a lot of layouts to happen and will
consequently run really slowly. The following code shows an example of
this. It contains two different programs that build up a line of _X_
characters 2,000 pixels wide and measures the time each one takes.

```{lang: "text/html", test: nonumbers}
<p><span id="one"></span></p>
<p><span id="two"></span></p>

<script>
  function time(name, action) {
    var start = Date.now(); // Current time in milliseconds
    action();
    console.log(name, "took", Date.now() - start, "ms");
  }

  time("naive", function() {
    var target = document.getElementById("one");
    while (target.offsetWidth < 2000)
      target.appendChild(document.createTextNode("X"));
  });
  // → naive took 32 ms

  time("clever", function() {
    var target = document.getElementById("two");
    target.appendChild(document.createTextNode("XXXXX"));
    var total = Math.ceil(2000 / (target.offsetWidth / 5));
    for (var i = 5; i < total; i++)
      target.appendChild(document.createTextNode("X"));
  });
  // → clever took 1 ms
</script>
```

## Styling

{{index "block element", "inline element", style, "strong (HTML tag)", "a (HTML tag)", underline}}

We have seen that different
HTML elements display different behavior. Some are displayed as
blocks, others inline. Some add styling, such as `<strong>` making its
content ((bold)) and `<a>` making it blue and underlining it.

{{index "img (HTML tag)", "default behavior", "style attribute"}}

The way
an `<img>` tag shows an image or an `<a>` tag causes a link to be
followed when it is clicked is strongly tied to the element type. But
the default styling associated with an element, such as the text color
or underline, can be changed by us. Here is an example using the `style`
property:

```{lang: "text/html"}
<p><a href=".">Normal link</a></p>
<p><a href="." style="color: green">Green link</a></p>
```

{{if book

The second link will be green instead of the default link color.

{{figure {url: "img/colored-links.png", alt: "A normal and a green link",width: "2.2cm"}}}

if}}

{{index "border (CSS)", "color (CSS)", CSS, "colon character"}}

A
style attribute may contain one or more _((declaration))s_, which are
a property (such as `color`) followed by a colon and a value (such as
`green`). When there is more than one declaration, they must be
separated by ((semicolon))s, as in `"color: red; border: none"`.

{{index "display (CSS)", layout}}

There are a lot of aspects that can be
influenced by styling. For example, the `display` property controls
whether an element is displayed as a block or an inline element.

```{lang: "text/html"}
This text is displayed <strong>inline</strong>,
<strong style="display: block">as a block</strong>, and
<strong style="display: none">not at all</strong>.
```

{{index "hidden element"}}

The `block` tag will end up on its own line since
((block element))s are not displayed inline with the text around them.
The last tag is not displayed at all—`display: none` prevents an
element from showing up on the screen. This is a way to hide elements.
It is often preferable to removing them from the document
entirely because it makes it easy to reveal them again at a later time.

{{if book

{{figure {url: "img/display.png", alt: "Different display styles",width: "4cm"}}}

if}}

{{index "color (CSS)", "style attribute"}}

JavaScript code can directly
manipulate the style of an element through the node's `style`
property. This property holds an object that has properties for all
possible style properties. The values of these properties are strings,
which we can write to in order to change a particular aspect of the
element's style.

```{lang: "text/html"}
<p id="para" style="color: purple">
  Pretty text
</p>

<script>
  var para = document.getElementById("para");
  console.log(para.style.color);
  para.style.color = "magenta";
</script>
```

{{index "camel case", capitalization, "dash character", "font-family (CSS)"}}

Some style property names contain dashes, such as `font-family`.
Because such property names are awkward to work with in JavaScript
(you'd have to say `style["font-family"]`), the property names in the
`style` object for such properties have their dashes removed and the
letters that follow them capitalized (`style.fontFamily`).

## Cascading styles

{{index "rule (CSS)", "style (HTML tag)"}}

{{indexsee "Cascading Style Sheets", CSS}}

The styling system for HTML is called ((CSS))
for _Cascading Style Sheets_. A _((style sheet))_ is a set of
rules for how to style elements in a document. It can be given
inside a `<style>` tag.

```{lang: "text/html"}
<style>
  strong {
    font-style: italic;
    color: gray;
  }
</style>
<p>Now <strong>strong text</strong> is italic and gray.</p>
```

{{index "rule (CSS)", "font-weight (CSS)", overlay}}

The _((cascading))_ in the name
refers to the fact that multiple such rules are combined to
produce the final style for an element. In the previous example, the
default styling for `<strong>` tags, which gives them `font-weight:
bold`, is overlaid by the rule in the `<style>` tag, which adds
`font-style` and `color`.

{{index "style (HTML tag)", "style attribute"}}

When multiple rules define
a value for the same property, the most recently read rule gets a
higher ((precedence)) and wins. So if the rule in the `<style>`
tag included `font-weight: normal`, conflicting with the default
`font-weight` rule, the text would be normal, _not_ bold. Styles in a
`style` attribute applied directly to the node have the highest
precedence and always win.

{{index uniqueness, "class attribute", "id attribute"}}

It is possible
to target things other than ((tag)) names in CSS rules. A rule for
`.abc` applies to all elements with `"abc"` in their class attributes.
A rule for `#xyz` applies to the element with an `id` attribute of
`"xyz"` (which should be unique within the document).

```{lang: "text/css"}
.subtle {
  color: gray;
  font-size: 80%;
}
#header {
  background: blue;
  color: white;
}
/* p elements, with classes a and b, and id main */
p.a.b#main {
  margin-bottom: 20px;
}
```

{{index "rule (CSS)"}}

The ((precedence)) rule favoring the most recently defined rule
holds true only when the rules have the same _((specificity))_. A rule's
specificity is a measure of how precisely it describes matching
elements, determined by the number and kind (tag, class, or ID) of
element aspects it requires. For example, a rule that targets `p.a` is more specific than
rules that target `p` or just `.a`, and would thus take precedence
over them.

{{index "direct child node"}}

The notation `p > a {…}` applies the given
styles to all `<a>` tags that are direct children of `<p>` tags.
Similarly, `p a {…}` applies to all `<a>` tags inside `<p>` tags,
whether they are direct or indirect children.

## Query selectors

{{index complexity}}

We won't be using ((style sheet))s all that much in
this book. Although understanding them is crucial to programming in
the browser, properly explaining all the properties they support and the
interaction among those properties would take two or three books.

{{index "domain-specific language"}}

The main reason I introduced
_((selector))_ syntax--the notation used in style sheets to determine
which elements a set of styles apply to—is that we can use this same
mini-language as an effective way to find ((DOM)) elements.

{{index "querySelectorAll method"}}

The `querySelectorAll` method, which is defined
both on the `document` object and on element nodes, takes a selector
string and returns an ((array-like object)) containing all the
elements that it matches.

```{lang: "text/html"}
<p>And if you go chasing
  <span class="animal">rabbits</span></p>
<p>And you know you're going to fall</p>
<p>Tell 'em a <span class="character">hookah smoking
  <span class="animal">caterpillar</span></span></p>
<p>Has given you the call</p>

<script>
  function count(selector) {
    return document.querySelectorAll(selector).length;
  }
  console.log(count("p"));           // All <p> elements
  // → 4
  console.log(count(".animal"));     // Class animal
  // → 2
  console.log(count("p .animal"));   // Animal inside of <p>
  // → 2
  console.log(count("p > .animal")); // Direct child of <p>
  // → 1
</script>
```

{{index "live data structure"}}

Unlike methods such as `getElementsByTagName`,
the object returned by `querySelectorAll` is _not_ live. It won't
change when you change the document.

{{index "querySelector method"}}

The `querySelector` method (without the
`All` part) works in a similar way. This one is useful if you want a
specific, single element. It will return only the first matching
element or null if no elements match.

{{id animation}}
## Positioning and animating

{{index "position (CSS)", "relative positioning", "top (CSS)", "left (CSS)", "absolute positioning"}}

The `position` style property
influences layout in a powerful way. By default it has a value of
`static`, meaning the element sits in its normal place in the
document. When it is set to `relative`, the element still takes up
space in the document, but now the `top` and `left` style properties
can be used to move it relative to its normal place. When `position`
is set to `absolute`, the element is removed from the normal document
flow—that is, it no longer takes up space and may overlap with other
elements. Also, its `top` and `left` properties can be used to
absolutely position it relative to the top-left corner of the nearest
enclosing element whose `position` property isn't `static`, or
relative to the document if no such enclosing element exists.

We can use this to create an ((animation)). The following document 
displays a picture of a cat that floats around in an ((ellipse)):

```{lang: "text/html"}
<p style="text-align: center">
  <img src="img/cat.png" style="position: relative">
</p>
<script>
  var cat = document.querySelector("img");
  var angle = 0, lastTime = null;
  function animate(time) {
    if (lastTime != null)
      angle += (time - lastTime) * 0.001;
    lastTime = time;
    cat.style.top = (Math.sin(angle) * 20) + "px";
    cat.style.left = (Math.cos(angle) * 200) + "px";
    requestAnimationFrame(animate);
  }
  requestAnimationFrame(animate);
</script>
```

{{if book

The gray arrow shows the path along which the image moves.

{{figure {url: "img/cat-animation.png", alt: "A moving cat head",width: "8cm"}}}

if}}

{{index "top (CSS)", "left (CSS)", centering, "relative positioning"}}

The picture is centered on the page and given a
`position` of `relative`. We'll repeatedly update that picture's `top`
and `left` styles in order to move it.

{{index "requestAnimationFrame function", drawing, animation}}

{{id animationFrame}}
The
script uses `requestAnimationFrame` to schedule the `animate` function
to run whenever the browser is ready to repaint the screen. The
`animate` function itself again calls `requestAnimationFrame` to
schedule the next update. When the browser window (or tab) is active,
this will cause updates to happen at a rate of about 60 per second,
which tends to produce a good-looking animation.

{{index timeline, blocking}}

If we just updated the DOM in a loop, the
page would freeze and nothing would show up on the screen. Browsers do
not update their display while a JavaScript program is running, nor do
they allow any interaction with the page. This is why we need
_requestAnimationFrame_—it lets the browser know that we are done
for now, and it can go ahead and do the things that browsers do, such
as updating the screen and responding to user actions.

{{index "smooth animation"}}

Our ((animation)) function is passed the current
((time)) as an argument, which it compares to the time it saw before (the
`lastTime` variable) to ensure the motion of the cat per millisecond
is stable, and the animation moves smoothly. If it just moved a fixed
amount per step, the motion would stutter if, for example, another
heavy task running on the same computer were to prevent the function
from running for a fraction of a second.

{{index "Math.cos function", "Math.sin function", cosine, sine, trigonometry}}

{{id sin_cos}}
Moving in
((circle))s is done using the trigonometry functions `Math.cos` and
`Math.sin`. For those of you who aren't familiar with these, I'll
briefly introduce them since we will occasionally need them in this
book.

{{index coordinates, pi}}

`Math.cos` and `Math.sin` are useful for
finding points that lie on a circle around point (0,0) with a radius
of one unit. Both functions interpret their argument as the position
on this circle, with zero denoting the point on the far right of the
circle, going clockwise until 2π (about 6.28) has taken us around the
whole circle. `Math.cos` tells you the x-coordinate of the point that
corresponds to the given position around the circle, while `Math.sin`
yields the y-coordinate. Positions (or angles) greater than 2π or less than
0 are valid—the rotation repeats so that _a_+2π refers to the same
((angle)) as _a_.

{{figure {url: "img/cos_sin.svg", alt: "Using cosine and sine to compute coordinates",width: "6cm"}}}

{{index "counter variable", "Math.sin function", "top (CSS)", "Math.cos function", "left (CSS)", ellipse}}

The cat
animation code keeps a counter, `angle`, for the current angle of the
animation and increments it in proportion to the elapsed time every
time the `animate` function is called. It can then use this angle to
compute the current position of the image element. The `top` style is
computed with `Math.sin` and multiplied by 20, which is the vertical
radius of our circle. The `left` style is based on `Math.cos` and
multiplied by 200 so that the circle is much wider than it is high,
resulting in an elliptic motion.

{{index "unit (CSS)"}}

Note that styles usually need _units_. In this case,
we have to append `"px"` to the number to tell the browser we are
counting in ((pixel))s (as opposed to centimeters, “ems”, or other
units). This is easy to forget. Using numbers without units will
result in your style being ignored—unless the number is 0, which
always means the same thing, regardless of its unit.

## Summary

JavaScript programs may inspect and interfere with the current
document that a browser is displaying through a data structure called
the DOM. This data structure represents the browser's model of the
document, and a JavaScript program can modify it to change the visible
document.

The DOM is organized like a tree, in which elements are arranged
hierarchically according to the structure of the document. The objects
representing elements have properties such as `parentNode` and
`childNodes`, which can be used to navigate through this tree.

The way a document is displayed can be influenced by _styling_, both
by attaching styles to nodes directly and by defining rules that
match certain nodes. There are many different style properties, such as
`color` or `display`. JavaScript can manipulate an
element's style directly through its `style` property.

## Exercises

{{id exercise_table}}
### Build a table

{{index "table (HTML tag)"}}

We built plaintext ((table))s in
[Chapter ?](object#tables). HTML makes laying out tables
quite a bit easier. An ((HTML)) table is built with the following tag
structure:

```{lang: "text/html"}
<table>
  <tr>
    <th>name</th>
    <th>height</th>
    <th>country</th>
  </tr>
  <tr>
    <td>Kilimanjaro</td>
    <td>5895</td>
    <td>Tanzania</td>
  </tr>
</table>
```

{{index "tr (HTML tag)", "th (HTML tag)", "td (HTML tag)"}}

For each
_((row))_, the `<table>` tag contains a `<tr>` tag. Inside of these `<tr>` tags,
we can put cell elements: either heading cells (`<th>`) or regular
cells (`<td>`).

{{index download, "MOUNTAINS data set", "table example"}}

The same
source data that was used in [Chapter ?](object#mountains)
is again available in the `MOUNTAINS` variable in the sandbox. It can also be http://eloquentjavascript.net/code/mountains.js[downloaded]
from the website[(http://eloquentjavascript.net/code#13[_eloquentjavascript.net/code#13_])]{if book}.

Write a function `buildTable` that, given an array of objects that all
have the same set of properties, builds up a DOM structure
representing a table. The table should have a header row with the
property names wrapped in `<th>` elements and should have one subsequent row per
object in the array, with its property values in `<td>` elements.

{{index "Object.keys function"}}

The `Object.keys` function, which returns an
array containing the property names that an object has, will probably
be helpful here.

{{index "right-aligning", "text-align (CSS)"}}

Once you have the basics
working, right-align cells containing numbers by setting their
`style.textAlign` property to `"right"`.

{{if interactive

```{lang: "text/html", test: no}
<style>
  /* Defines a cleaner look for tables */
  table  { border-collapse: collapse; }
  td, th { border: 1px solid black; padding: 3px 8px; }
  th     { text-align: left; }
</style>

<script>
  function buildTable(data) {
    // Your code here.
  }

  document.body.appendChild(buildTable(MOUNTAINS));
</script>
```
if}}

{{hint

{{index "createElement method", "table example", "appendChild method"}}

Use `document.createElement` to create new element nodes,
`document.createTextNode` to create text nodes, and the `appendChild`
method to put nodes into other nodes.

You should loop over the key names once to fill in the top row and
then again for each object in the array to construct the data
rows.

Don't forget to return the enclosing `<table>` element at the end of
the function.

hint}}

### Elements by tag name

{{index "getElementsByTagName method", recursion}}

The
`getElementsByTagName` method returns all child elements with a given
tag name. Implement your own version of it as a regular nonmethod
function that takes a node and a string (the tag name) as arguments
and returns an array containing all descendant element nodes with the
given tag name.

{{index "tagName property", capitalization, "toLowerCase method", "toUpperCase method"}}

To find the tag name of an element,
use its `tagName` property. But note that this will return the tag
name in all uppercase. Use the `toLowerCase` or `toUpperCase` string
method to compensate for this.

{{if interactive

```{lang: "text/html", test: no}
<h1>Heading with a <span>span</span> element.</h1>
<p>A paragraph with <span>one</span>, <span>two</span>
  spans.</p>

<script>
  function byTagName(node, tagName) {
    // Your code here.
  }

  console.log(byTagName(document.body, "h1").length);
  // → 1
  console.log(byTagName(document.body, "span").length);
  // → 3
  var para = document.querySelector("p");
  console.log(byTagName(para, "span").length);
  // → 2
</script>
```
if}}

{{hint

{{index "getElementsByTagName method", recursion}}

The solution is most
easily expressed with a recursive function, similar to the
[`talksAbout` function](dom#talksAbout) defined earlier in
this chapter.

{{index concatenation, "concat method", closure}}

You could call
`byTagname` itself recursively, concatenating the resulting arrays to
produce the output. For a more efficient approach, define an inner
function that calls itself recursively and that has access to an
array variable defined in the outer function to which it can add the
matching elements it finds. Don't forget to call the ((inner
function)) once from the outer function.

{{index "nodeType property", "ELEMENT_NODE code"}}

The recursive function
must check the node type. Here we are interested only in node type 1
(`document.ELEMENT_NODE`). For such nodes, we must loop over their
children and, for each child, see whether the child matches the query while also doing
a recursive call on it to inspect its own children.

hint}}

### The cat's hat

{{index "cat's hat (exercise)"}}

Extend the cat ((animation)) defined
[earlier](dom#animation) so that both the cat and his hat
(`<img src="img/hat.png">`) orbit at opposite sides of the ellipse.

Or make the hat circle around the cat. Or alter the animation in some
other interesting way.

{{index "absolute positioning", "top (CSS)", "left (CSS)", "position (CSS)"}}

To make positioning multiple objects easier, it is probably a
good idea to switch to absolute positioning. This means that `top` and
`left` are counted relative to the top left of the document. To avoid
using negative coordinates, you can simply add a fixed number of
pixels to the position values.

{{if interactive

```{lang: "text/html", test: no}
<img src="img/cat.png" id="cat" style="position: absolute">
<img src="img/hat.png" id="hat" style="position: absolute">

<script>
  var cat = document.querySelector("#cat");
  var hat = document.querySelector("#hat");
  // Your code here.
</script>
```

if}}
