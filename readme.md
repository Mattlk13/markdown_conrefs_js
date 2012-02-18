# Introduction

I wrote technical documentation in my formative years using [the DITA standard](http://en.wikipedia.org/wiki/Darwin_Information_Typing_Architecture). Nowadays, I write documentation in Markdown, but one of the things I _really_ miss is the ability to do content references&mdash;or _conrefs_&mdash;from one file to another.

The idea is simple: if you have a term, phrase, or block that you're reusing throughout the documentation, it's best to just define it once, tag it with a unique ID, then refer to that ID whenever you want to pull the content in. Simple, and much more efficient then search and replace.

I wrote a module for this functionality in Node.js, and it would be my dream of dreams if other Markdown packages for other languages implement similar functionality.

# Installation

From npm, just do

    npm install markdown_conrefs

# Syntax

The syntax follows the superior [Maruku metadata format for Markdown](http://maruku.rubyforge.org/maruku.html#meta). IDs are either attached at the block level, or inline.

To attach to the block level, just create metadata for the last line of the block, _with an id element_, like so:

```
* Item 1
* Item 2
* Item 3
{: #aNiceList}

Here's a great, and final, paragraph.
{: id="myPara"}
```

Notice that there are three ways in Maruku to define attributes; this conref system supports all of them.

To attach conrefs inline, you'll need to wrap the content with brackets (`[ ]`), and then continue with the same attribute format, like so:

```
I don't want to have to rewrite this.]{: .class1 id=aTruth .class2}
[**A RELIEF**]{: key=value id='expression' foo=bar}
[Genesis]{: #newProj} 
```

Note: this format differs slightly from Maruku. Maruku metadata only applies to Markdown inline elements, like bold, or italic. This system allows you to conref even plain-text. In fact, [namp](https://github.com/gjtorikian/namp), my Markdown processor written in Node.js, also supports this notation.

**Remember**: the syntax to define this metadata is `curly brace-colon-space`! Why is that important? Well...

In order to reference the conref, you'll just use this syntax wherever you want to do an insertion: `{:id}`. That's `curly brace-colon-id-curly brace`--nothing else! It's irrelevant if you defined your id with `#`, `id=""`, or `id=''`--all you want to do is use the id name. To use the examples above, that'd be:

```
{:aNiceList}
{:myPara}

{:aTruth}
{:expression}
{:newProj}
```

## Pulling additional attributes
Any other attributes you've defined--class names, language identifiers for fenced code blocks, _e.t.c._--get pulled into the resulting document as well, **but the ID is stripped**. 

Why? Consider the following text:

```
I am working on [Project X]{: #product .secret}. I love being on {:product}. It's more rewarding to be a part of {:product}.
```

If IDs were kept, the block would resolve as:

```
I am working on [Project X]{: #product .secret}. I love being on [Project X]{: #product .secret}. It's more rewarding to be a part of [Project X]{: #product .secret}.
```

That's now three elements in the same document with the same ID, `product`. Instead, the resolve happens like this:

```
I am working on [Project X]{: #product .secret}. I love being on [Project X]{: .secret}. It's more rewarding to be a part of [Project X]{: .secret}.
```

# Using

First, add `require('markdown_conrefs')` to your code. This module only has two functions:

* `init(source, type [, exclusions ])` must be called first! This creates the id-to-content hash. The parameters are:
  * `source` is either:  
   * A string containing the path to a single file
   * A string name of a directory
   * An Array of strings for directories. 
   `source` represents the file you want to parse, _or_, the highest level directory you want to start searching content references for--this module will recurisvely find all conref IDs in files to keep track of them. **Note**: if you provide just the name of a single file for `source`, the other two parameters are unnecessary. In fact, don't pass anything, because it won't work.
  * `type` is the extension of your markdown files. 
  * `exclusions` is an array of strings, indicating any files or directories you don't want to process when `source` is a directory. This is optional.

This function has no return value, and is synchronous/blocking.

* `replaceConref(data)` when you're ready to apply the conrefs, use this function. `data` is the string containing markdown refrences--it's usually the file you're reading. This function returns the string with the conref references replaced.

Check out the _test/_ directory for an example; just run `cd test && node test/test.js` from this directory.

Keep in mind that this module only replaces the references; you'll still need to run a Markdown parser in order to actually generate HTML.

## Error Checking

The module automatically halts if:

* You have more than one ID with the same reference
* You refer to an ID that doesn't exist

# Benchmarks

I tested this module by creating a single file with 10,000 conref IDs, and 10,000 files with a single conref ID. Presumably, lookups would cost the same amount of time, so I made a single call to `markdown_conref.init()` on each set of files.

On average, the single file took 7ms to complete; the multiple files took 450ms. In other words, they're both damned fast, despite my mediocre programming. For the technical writer, a single file might be easier to manage.