# Liquid (OrchardCore.Liquid)

This module provides a way to create templates securely from the admin site.
For more information about the Liquid syntax, please refer to this site: https://shopify.github.io/liquid/

## General concepts

### HTML escaping

All outputs are encoded into HTML by default. It means that any string that returns some HTML reserved chars will
be converted to the corresponding HTML entities. If you need to render some raw HTML chars you can use the `Raw` filter.

## Content Item Filters

All the default filters that are available in the standard Liquid syntax are available in OrchardCore. On top of that each Orchard module can
provide custom filters for their own purpose. Here is a list of common filters that apply to content items.

### display_url

Returns the url of the content item

Input
```
{{ ContentItem | display_url }}
```

Output
```
/blog/my-blog-post
```

### display_text

Returns the title of the content item

Input
```
{{ ContentItem | display_text }}
```

Output
```
My Blog Post
```

### slugify

Convert a text into a string that can be used in a url.

Input
```
{{ "This is some text" | slugify }}
```

Output
```
this-is-some-text
```

### container

Returns the container content item of another content item.


Input
```
{{ ContentItem | container | display_text }}
```
In this example we assume `ContentItem` represents a blog post.

Output
```
Blog
```

### local

Converts a UTC date and time to the local date and time based on the site settings.

Input
```
{{ "now" | local | date: "%c" }}
or
{{ ContentItem.CreatedUtc | local | date: "%c" }}
```

Output
```
Wednesday, 02 August 2017 11:54:48
```

### t

Localizes a string using the current culture.

Input
```
{{ "Hello!" | t }}
```

Output
```
Bonjour!
```

## Properties

By default the liquid templates have access to a common set of objects.

### ContentItem

Represents the current content item that contains the liquid template.

The following properties are available on the `ContentItem` object.

| Property | Example | Description |
| --------- | ---- |------------ |
| `Id` | `12` | The id of the document in the database |
| `ContentItemId` | `4qs7mv9xc4ttg5ktm61qj9dy5d` | The common identifier of all versions of the content item |
| `ContentItemVersionId` | `4jp895achc3hj1qy7xq8f10nmv` | The unique identifier of the content item version |
| `Number` | `6` | The version number |
| `Owner` | `admin` | The username of the creator of this content item |
| `Author` | `admin` | The username of the editor of this version |
| `Published` | `true` | Whether this content item version is published or not |
| `Latest` | `true` | Whether this content item version is the latest of the content item |
| `ContentType` | `BlogPost` | The content type |
| `CreatedUtc` | `2017-05-25 00:27:22.647` | When the content item was first created or first published |
| `ModifiedUtc` | `2017-05-25 00:27:22.647` | When the content item version was created |
| `PublishedUtc` | `2017-05-25 00:27:22.647` | When the content item was last published |
| `Content` | `{ ... }` | A document containing all the content properties. See specific documentation for usage .|

#### Content property

The `Content` property of a content item exposes all its elements, like parts and fields. It is possible to
inspect all the available properties by evaluating `Content` directly. It will then render the full document.

The convention is that each Part is exposed by its name as the first level.
If the content item has custom fields, they will be available under a part whose name will match the content type.

For example, assuming the type `Product` has a Text field named `Size`, access the value of this field for a 
content item would be:

```
{{ ContentItem.Content.Product.Size.Text }}
```

Similarly, if the content item has a `Title` part, we can access it like this:

```
{{ ContentItem.Content.TitlePart.Title }}
```

### User

Represents the authenticated user for the current request.

The following properties are available on the `User` object.

| Property | Example | Description |
| --------- | ---- |------------ |
| `Identity.Name` | `admin` | The name of the authenticated user |

## Shape Filters

These filters let you create, filter and display shapes.

### new_shape

Returns a shape with the specified name as input.

Input
```
{% assign date_time = "DateTime" | new_shape %}
```

### shape_string

Renders a shape to a string value.

Input
```
{{ "DateTime" | new_shape | shape_string }}

```

Output
```
Monday, September 11, 2017 3:29:26 PM
```

### clear_alternates

Removes any alternates from an input shape.

Input
```
{{ my_shape | clear_alternates }}

```

### add_alternates

Adds alternates to an input shape.

Input
```
{{ my_shape | add_alternates: "alternate1 alternate2" }}

```

### clear_classes

Removes any classes from an input shape.

Input
```
{{ my_shape | clear_classes }}

```

### add_classes

Adds classes to an input shape.

Input
```
{{ my_shape | add_classes: "class1 class2" }}

```

### shape_type

Replaces the shape type of an input shape.

Input
```
{{ my_shape | shape_type: "OtherShapeType" }}

```

### display_type

Replaces the display type of an input shape.

Input
```
{{ my_shape | display_type: "Summary" }}

```

### shape_position

Replaces the position of an input shape.

Input
```
{{ my_shape | shape_position: "Content:before" }}

```

### shape_tab

Replaces the tab of an input shape.
Input
```
{{ my_shape | shape_tab: "properties" }}

```

### remove_item

Removes a named shape from an input shape items.

Input
```
{% display Model.Content | remove_item: "BodyPart" %}
```

In this example, the `Model.Content` property evaluates to a zone shape, typically from a Content Item shape template, which contains the `BodyPart` shape
rendered for the Body Part element. This call will remove the specific shape named `BodyPart`.

### set_properties

Replaces a property of an input shape.

```
{{ Model.Pager | set_properties: next_class: 'next', next_text: '>>' }}
```

## Layout Tags

### render_body

In a layout, renders the body of the current view.

Input
```
{% render_body %}
```

### render_section

In a layout, renders the section with the specified name.

Input
```
{% render_section "Header", required: false %}
```

### page_title

Alters and renders the title of the current page.

Input
```
{% page_title Site.SiteName, position: "before", separator: " - " %}
```

The default parameter is a text that is appended to the current value of the title.
`position` is where the value is appended, in this example at the beginning.
`separator` is a string that is used to separate all the fragments of the title.

## Shape Tags

### display

Renders a shape. Similar to the `shape_string` filter.

Input
```
{% assign date_time = "DateTime" | new_shape %}
{% display date_time %}
```

Output
```
Monday, September 11, 2017 3:29:26 PM
```

### build_display

Creates the display shape for a content item. It can be used in conjunction with `display` 
to render a content item.

Input
```
{% display mycontentitem | build_display: "Detail"  %}
```

### shape

Renders a specific named tag with its properties

Input
```
{% shape "menu", alias: "alias:main-menu", cache_id: "main-menu", cache_duration: "00:05:00", cache_tag: "alias:main-menu" %}
```

### zone

Renders some HTML content in the specified zone.

Input
```
{% zone "Header" %}
    <!-- some content goes here -->
{% endzone %}
```

The content of this block can then be reused from the Layout using the `{% display Model.Header %}` code.

## Tag Helper tags

ASP.NET Core MVC provides a set of tag helpers to render predefined html outputs. The Liquid module provides a way to call into these Tag Helpers using custom liquid tags.

### link

Invokes the `link` tag helper from the **Orchard.ResourceManagement** package.

### meta

Invokes the `meta` tag helper from the **Orchard.ResourceManagement** package.

### resources

Invokes the `resources` tag helper from the **Orchard.ResourceManagement** package.

### script

Invokes the `script` tag helper from the **Orchard.ResourceManagement** package.

### style

Invokes the `style` tag helper from the **Orchard.ResourceManagement** package.

### a

Invokes the `a` tag helper from the MVC package.

## CREDITS

### Fluid

https://github.com/sebastienros/fluid
Copyright (c) 2017 Sebastien Ros
MIT License
