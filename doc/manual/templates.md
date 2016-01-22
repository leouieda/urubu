---
title: Templates
layout: page 
pager: true
author: Jan Decaluwe
---

Overview
========

Templates define the html layout for a particular page type.  In a template you
can mix plain html with control structures and variable interpolation. The html
page is generated by evaluating the template with the appropriate evaluation
context, provided by Urubu.

Templates are part of the website project setup. If you are a content
contributor, you may not have to worry about them.  All you have to do is
specify the appropriate layout name in the YAML front matter of your content
files.

In general, it is better to do programming in Python code.  However, for the
purpose of generation of pages in a format such as html, this is not very
practical.  Therefore, a template contains both html and programming
constructs. Template programming is good for common tasks like the following:

* iteration over a list of items
* testing whether an item is defined
* filtering items, possibly with user-defined filters 
* comparing the loop item object with the current object, to check whether it is active

Urubu interacts with templates by providing an evaluation context with the
appropriate objects.  The goal is to make the job of the templates as easy as
possible with ready-to-use object attributes. 

The template library
====================

Urubu uses the [jinja2] templating language library.

Jinja2 has great [documentation][jinja2_docs] and you should
check it out when using templates in Urubu.

[jinja2_docs]: http://jinja2.pocoo.org/docs

A great feature of Jinja2 is template inheritance. With this technique,
you can easily generate small variations of a parent template.

Link objects
============

Description
-----------

Link objects are the primary objects that you use in templates. 
They come in a number of flavours: 

Link object        | Description
-------------------|---------------
global link object | Defined in the `_site.yml` file.
local link object  | Defined locally in the `content` attribute of an index file.
folder object      | Corresponds to a project subdirectory.
page object        | Corresponds to a Markdown content file.
tag folder object  | Dedicated folder for content ordered by tag. 
tag object         | Represents content corresponding to a specific tag. 

Attributes
----------

Link objects have attributes. The defined attributes depend on the type of the
link object. 

The following attributes are common to **all link objects**:

Attribute      | Description 
---------------|---------------------------
`url`          | The url of the object. 
`title`        | The title of the object.

**All link objects except local link objects** also
have an `id` attribute:

Attribute      | Description 
---------------|---------------------------
`id`           | The unique id by which the object is known in the project. 

**Folder and page objects** have the following attributes: 

Attribute      | Description 
---------------|---------------------------
`fn`           | The pathname of the file or directory corresponding the object. 
`components`   | The components of the object's pathname, without file extension, as a list.
`mdate`        | Modification date

**Folder objects** also have a `content` attribute: 

Attribute      | Description 
---------------|---------------------------
`content`      | The content of the folder as a list of page & folder objects.

In addition, all attributes specified in the YAML front matter
of the corresponding index file will be available as attributes of
the folder object.

**Page objects** have the following additional attributes:

Attribute      | Description 
---------------|---------------------------
`layout`       | The template to render the object as a html file. 
`body`         | The page content in html.
`toc`          | The table of contents of the page as an unordered html list.
`breadcrumbs`  | Breadcrumbs as a list. The current page object is at position 0, the containing folder objects are at the higher positions.
`prev`         | The previous page object in the content, or `None` if there is none
`next`         | The next page object in the content, or `None` if there is none

In addition, all attributes specified in the YAML front matter of the
corresponding content file are available as attributes of the page object.

Index pages
-----------

Index pages are associated with `index.md` files. They are special in the sense
that they define the attributes and the content of a folder. Therefore, they
have the same `content` attribute as the corresponding folder object.

Tag objects
-----------

Tag objects are inferred by Urubu automatically. They list the content
corresponding to a tag.

{% raw %}
Attribute      | Description 
---------------|---------------------------
`id`           | `/tag/{{tag}}`
`components`   | `[tag, {{tag}}]`
`title`        | `tag`
`tag`          | `tag`
`layout`       | `tag`  
`content`      | List of page & folder objects corresponding to `tag`. 
{% endraw %}

The tag content is ordered by date, most recent first. If the date is not
defined, the modification date is used as a fallback (`mdate` attribute).

The layout name is predefined to `tag`. You have to provide the `tag.html`
template to trigger the rendering of tag objects. In the simplest case, it
may be sufficient to inherit from a general index layout.

Tag folder object
-----------------

The tag folder object is a special top-level folder whose `id` is `/tag`. Urubu
infers tag-related content for this folder automatically.

You can optionally create the corresponding directory in the source code, and
use the index file to set attributes such as the `layout`. In any case, Urubu
will create the object if tags are used, and infer the `content` attribute. 

Attribute      | Description 
---------------|---------------------------
`id`           | `/tag` 
`components`   | `[tag]`
`content`      | A list of tag objects, inferred by Urubu. 

The content is ordered according to the content size of tag objects,
the largest one first.


Context variables
=================

Urubu makes the context available to templates with
two context variables.

`site` 
------

This variable holds site-wide information.

It has one predefined attribute:

Attribute      | Description 
---------------|---------------------------
`reflinks`     | A mapping from all reference ids to link objects.

Note that the id of the root object is `/`. Starting from there,
you can traverse the whole site.

In addition, all the attributes specified in the `_site.yml` file
will be available as attributes of the `site` variable.

`this`
------

This variable holds the current page or tag object.
