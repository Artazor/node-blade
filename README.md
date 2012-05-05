Blade - HTML Template Compiler
==============================

Blade is a HTML Template Compiler, inspired by Jade &amp; Haml, implemented in
JavaScript, so it will run on your microwave oven.

Never write HTML again. Please.

Table of Contents
-----------------

- [Features](#features)
- [Todo](#todo)
- [Installation](#installation)
- [Language Syntax](#syntax)
- [API](#api)
- [Implementation Details](#implementation-details)
- [License](#license)

Features
--------

- Write extremely readable short-hand HTML
- Insert escaped and unescaped text and vanilla JavaScript code
- Code and text are escaped by default for security/convenience
- Functions (like Jade mixins)
- Dynamic file includes
- Regular blocks and Parameterized blocks
- True client-side template support with caching, etc.
- Supports Express.JS
- HTML Comments and block comments
- Text filters

Blade does more than Jade, and it does less than Jade. Your Jade templates
will probably need some modifications before they will work with Blade.

Todo
----

Blade was implemented entirely in less than 4 days.
So, there is still stuff to do:

- Better error handling and error reporting (right now it kinda sux)
- Finish client-side runtime
- Change tag ending based on doctype
- Executable to compile and/or render templates via command line

Installation
------------

for Node (via npm): `npm install blade`

for Browsers:

Runtime only: `wget https://raw.github.com/bminer/node-blade/master/dist/blade-runtime.min.js`

Compiler + Runtime: `wget https://raw.github.com/bminer/node-blade/master/dist/blade.min.js`

Syntax
------

### Tags

Like Jade, a tag is simply a word. For example, the string `html` will render to `<html></html>`.

You can have 'id's:

```
div#awesome
```

which renders as `<div id="awesome"></div>`.

Any number of classes work, separated by a dot (`.`)

```
div.task-details.container
```

which renders as `<div class="task-details container"></div>`.

Tag attributes?  Yep, they work pretty much like Jade, too.
Put attributes in parenthesis, separate attributes with a comma, space, newline, or whatever.

`a(href="/homepage", onclick="return false;")` renders as:

```html
<a href="/homepage" onclick="return false;"></a>
```

You can also have line feeds or weird whitespace between attributes, just like in Jade.
Whatever. This works, for example:

```
input(
		type="text"
		name="email"
		value="Your email here"
	)
```

Yes... the `class` attribute is handled with extra special care. Pass an array or string. Yes,
classes (delimited by ".") from before will be merged with the value of the attribute.

`div#foo.bar.dummy(class="another dude")` renders as: `<div id="foo" class="bar dummy another dude"></div>`

div div div is annoying... so we can omit this if we specify an id or some classes:

```
#foo
.bar
#this.is.cool
```

renders as:

```html
<div id="foo"></div><div class="bar"></div><div id="this" class="is cool"></div>
```

Also, tags without matching ending tag like `<img>` render properly.

### Indenting

It works. You can indent with any number of spaces or with a single tab character.
Jade gives you a lot of weird indent flexibility. Blade, by design, does not.

```
html
	head
	body
		#content
```

renders as:

```html
<html><head></head><body><div id="content"></div></body></html>
```

### Text

It works, too. Simply place content after the tag like this:

```
p This text is "escaped" by default. Kinda neat.
```

renders as:

```html
<p>This text is &quot;escaped&quot; by default. Kinda neat.</p>
```

Want unescaped text?  Large blocks of text? Done.

```
p! This will be <strong>unescaped</strong> text.
	|
		How about a block? (this is "escaped", btw)
		Yep. It just works!
		Neato.
```

renders as:

```html
<p>This will be <strong>unescaped</strong> text.
How about a block? (this is &quot;escaped&quot;, btw)
Yep. It just works!
Neato.</p>
```

Rules are:

- Text is escaped by default
- Want unescaped text? Precede with a `!`
- Large text block? Use `|` and indent properly.
- Unescaped text block? Use `|!` or even just `!` works.

### Text filters

Need `<br/>` tags inserted? Use a built-in filter, perhaps?

```
p
	:nl2br
		How about some text with some breaks?
		
		Yep! It works!
```

renders as:

```html
<p>How about some text with some breaks?<br/><br/>Yep! It works!</p>
```

Built-in text filters include:

- :nl2br - Converts newline characters to `<br/>`
- :cdata - Surrounds text like this: `<![CDATA[` ...text goes here... `]]>`
	Text should not contain `]]>`.
- :markdown (must have [markdown-js](https://github.com/evilstreak/markdown-js) installed)
- :md (alias for :markdown)

Filters are essentially functions that accept a text string and return HTML. They
cannot modify the AST directly.

And, you can add custom filters at runtime using the API.

### Code

Use dash (`-`) to specify a code block.  Use equals (`=`) to specify code output.  A few examples, please?

Code blocks:

```
#taskStatus
	- if(task.completed)
		p You are done. Do more! >:O
	- else
		p Get to work, slave!
```

Code that outputs (i.e. in a text block or at the end of a tag).
It's just like a text block, except with an `=`.

```
#taskStatus= task.completed ? "Yay!" : "Awww... it's ok."
p
	| The task was due on
	= task.dueDate
```

When using code that outputs, the default is to escape all text. To turn off escaping, just
prepend a "!", as before:

```
p
	!= some_html
```

Extra "|" characters are okay, too.  Just don't forget that stuff after the "="
means JavaScript code!

```
p
	|= "escape me away & away"
```

renders `<p>escape me away &amp; away</p>`

#### Variable names to avoid

Blade, like other template engines, defines local variables within every single view. You
should avoid using these names in your view templates whenever possible:

- `locals`
- `runtime`
- `cb`
- `buf`
- `__` (that's two underscores)
- Any of the compiler options (i.e. `debug`, `minify`, etc.)
- `blade`

### Doctypes

Don't forget a doctype!  Actually, you can, whatever... defaults to HTML 5, of course.

Add a doctype using `doctype` keyword or `!!!` like this:

`!!! 5` means use HTML 5 doctype.

Use the list of built-in doctypes or pass your own like this:

```
doctype html PUBLIC "-//W3C//DTD XHTML Basic 1.1//EN
```

which renders as `<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML Basic 1.1//EN>`

Put the doctype at the top of your Blade files, please. Here is the list of built-in doctypes:

```javascript
var doctypes = exports.doctypes = {
  '5': '<!DOCTYPE html>',
  'xml': '<?xml version="1.0" encoding="utf-8" ?>',
  'default': '<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">',
  'transitional': '<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">',
  'strict': '<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">',
  'frameset': '<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Frameset//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-frameset.dtd">',
  '1.1': '<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">',
  'basic': '<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML Basic 1.1//EN" "http://www.w3.org/TR/xhtml-basic/xhtml-basic11.dtd">',
  'mobile': '<!DOCTYPE html PUBLIC "-//WAPFORUM//DTD XHTML Mobile 1.2//EN" "http://www.openmobilealliance.org/tech/DTD/xhtml-mobile12.dtd">'
};
```

Yes, you can modify the list of built-in doctypes through the API. Why would you, though?

### Comments

Use `//` for a line comment.  Use `//-` if you don't want the comment to be rendered.  Block comments work, too.

```
//Comment example 1
//-Comment example 2
//
	#wow
	p Block comments work, too
```

renders as:

```html
<!--Comment example 1--><!--<div id="wow"></div><p>Block comments work, too</p>-->
```

Conditional comments work like this:

```
head
	//if lt IE 8
		script(src="/ie-really-sux.js")
```

renders as:

```html
<head><!--[if lt IE 8]><script src="/ie-really-sux.js"></script><![endif]--></head>
```

### Functions

Functions are reusable mini-templates. They are similar to 'mixins' in Jade.

Defining a function:

```
function textbox(name, value)
	input(type="text", name=name, value=value)
```

Calling a function and inserting into template structure:

```
form
	!=functions.textbox("firstName", "Blake")
```

Or... maybe just putting the generated HTML into a variable?

```
- var text = functions.textbox("firstName", "Blake");
form
	!=text
```

Both examples would render:

```html
<form><input type="text" name="firstName" value="Blake"/></form>
```

Limitation: Don't use a local variable with the name `functions`... for obvious reasons.

### Dynamic file includes

`include "file.blade"`

This will dynamically (at runtime) insert "file.blade" right into the current view, as if it
was a single file

### Blocks

Blocks allow you to mark places in your template with code that may or may not be
rendered later.

You can do a lot with blocks, including template inheritance, etc. They behave quite
differently from Jade.

There are two types of blocks: regular blocks and parameterized blocks.

#### Regular blocks

Regular blocks are defined using the "block" keyword followed by a block name. Then,
you optionally put indented block content below. Like this:

```
block regular_block
	h1 Hello
	p This is a test
```

Assuming nothing else happens to the block, it will be rendered as
`<h1>Hello</h1><p>This is a test</p>` as expected. Empty blocks are also permitted.
A simple, empty block looks like this: `block block_name`

Of course, the purpose of declaring/defining a block is to possibly modify it later.
You can modify a block using three different commands:

- Use the `append` keyword to append to the matching block.
- Use the `prepend` keyword to prepend to the matching block.
- Use the `replace` keyword to replace to the matching block.

Example:

```
append regular_block
	p This is also a test
```

#### Replacing a block

Replacing a block is somewhat confusing, so I will explain further. If you replace
a block, you are not changing the location of the defined block; you are only
replacing the content of the block at its pre-defined location. If you want to change
the location of a block, simply re-define a new block (see below).

In addition, when you replace a block, all previously appended and prepended content is
lost. The behavior is usually desired, but it can sometimes be a source of confusion.

If you replace a parameterized block (described below), you cannot call "render" on
that block anymore.

At this time, you cannot replace a block with a parameterized block. The "replace"
command will not accept parameters.

#### Parameterized blocks

The other type of block is called a parameterized block, and it looks like this:

```
block regular_block(headerText, text)
	h1= headerText
	p= text
```

Parameterized blocks do not render automatically because they require parameters.
Therefore, assuming nothing else happens to the block, the block will not be rendered
at all.

To render a block, use the "render" keyword like this:

```
render regular_block("Some header text", 'Some "paragraph" text')
```

Now, assuming nothing else happens to the block, the block will be rendered as:

```html
<h1>Some header text</h1><p>Some &quot;paragraph&quot; text</p>
```

Parameterized blocks are really cool because "append", "prepend", and "replace"
all work, too. You don't need to "render" the block to use "append", "prepend", and
"replace"

Another example:

```
head
	block header(pageTitle)
		title= pageTitle
body
	h1 Hello
	render header("Page Title")
	append header
		script(src="text/javascript")
	prepend header
		meta
```

Will output:

```html
<head>
	<meta/>
	<title>Page Title</title>
	<script src="text/javascript"></script>
</head>
<body>
	<h1>Hello</h1>
</body>
```

If you do not insert a `render` call, the defined block will not render, and
all `append`, `prepend`, or `replace` will still work.

#### What happens if I use `block block_name` more than once for the same block name?

You can re-define a block that has already been defined with another "block"
statement. This completely destroys the previously defined block.
Previously executed "append", "prepend", "replace", and "render" blocks do not affect the
re-defined block.

In summary...

- Use the `block` keyword to mark where the block will go (block definition).
- Use the `render` keyword to render the matching "parameterized" block.
	Do not use this on a regular block.
- Use the `append` keyword to append to the matching block.
- Use the `prepend` keyword to prepend to the matching block.
- Use the `replace` keyword to replace the matching block.

### Template Inheritance

There is no `extends` keyword.  Just use blocks and includes:

layout.blade:

```
html
	head
		block title(pageTitle)
			title=pageTitle
	body
		block body
```

homepage.blade:

```
include "layout.blade"
render title("Homepage")
replace block body
	h1 Hello, World
```

If you render layout.blade, you get: `<html><head></head><body></body></html>`, but if you
render homepage.blade, you get:

```html
<html>
	<head>
		<title>Homepage</title>
	</head>
	<body>
		<h1>Hello, World</h1>
	</body>
</html>
```

API
---

`var blade = require('blade');`

### blade.compile(string, [options,] cb)

Asynchronously compiles a Blade template from a string.

- `string` is a string of Blade
- `options` include:
	- `filename` - the filename being compiled (required when using includes
		or the `cache` option)
	- `cache` - cache the compiled template
	- `debug` - outputs debug info to the console (defaults to false)
	- `minify` - if true, Blade generates a minified template without debugging
		information (defaults to false)
	- `doctypes` - use this Object instead of `blade.Compiler.doctypes`
	- `inlineTags` - use this Object instead of `blade.Compiler.inlineTags`
	- `filters` - use this Object instead of `blade.Compiler.filters`
- `cb` is a function of the form: `cb(err, fn)` where `err` contains
	any parse or compile errors and `fn` is the compiled template.
	If an error occurs, `err` may contain the following properties:
	- `message` - The error message
	- `expected` - If the error is a 'SyntaxError', this is an array of expected tokens
	- `found` - If the error is a 'SyntaxError', this is the token that was found
	- `filename` - The filename where the error occurred
	- `offset` - The offset in the string where the error occurred.
	- `line` - The line # where the error occurred
	- `column` - The column # where the error occurred

You can render a compiled template by calling the function: `tmpl(locals, cb)`
	- `locals` are the local variables to be passed to the view template
	- `cb` is a function of the form `function(err, html)` where `err` contains
		any runtime errors and `html` contains the rendered HTML.

In addition, a compiled template has these properties:
	- `template` - a function that also renders the template but accepts 3 parameters:
		`tmpl.template(locals, runtime, cb)`. This simply allows you to use a custom
		runtime environment, if you choose to do so.

You can call `tmpl.toString()`, just like you can on any other JavaScript function.
This might be useful for client-side templates, for example.

### blade.compileFile(filename, [options,] cb)

Asynchronously compile a Blade template from a filename on the filesystem.

- `filename` is the filename
- `options` - same as `blade.compile` above, except `filename` option is always
	overwritten	with the `filename` specified.
- `cb` - same as `blade.compile` above

### blade.renderFile(filename, options, cb)

Convenience function to asynchronously compile a template and render it.

- `filename` is the filename
- `options` - same as `blade.compileFile` above. This object is also passed
	directly to the view, so it should also contain your view's local variables.
- `cb` - a function of the form `function(err, html)`

### blade.Compiler

The compiler itself. It has some useful methods and properties.

### blade.Compiler.parse(string)

Just generates the parse tree for the string. For debugging purposes only.

```javascript
var blade = require('blade');
blade.compile("string of blade", options, function(err, tmpl) {
	tmpl(locals, function(err, html) {
		console.log(html);
	});
});
```

Implementation Details
----------------------

The Blade parser is built using [PEG.js](https://github.com/dmajda/pegjs).
Thanks to the PEG.js team for making this project much easier than I had
anticipated!

License
-------

See the LICENSE.txt file.
