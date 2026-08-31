---
icon: lucide/tags
tags:
  - Plugins
  - Tags
---

# Tags

The tags plugin assigns tags to pages and generates tag listings. It replaces
the built-in tags plugin in Material for MkDocs and supports hierarchical tags,
filtered listings, tag references, and listing entries in the table of
contents.

## Configuration

Enable tags with the default settings:

=== "`zensical.toml`"

    ``` toml
    [project.plugins."material/tags"]
    ```

=== "`mkdocs.yml`"

    ``` yaml
    plugins:
      - material/tags
    ```

Zensical also accepts `tags` and named instances such as
`material/tags/public`. In TOML projects, use the corresponding key under
`project.plugins`.

The plugin is enabled when it is configured. You can disable an instance with
`enabled = false` or `enabled: false`.

### Source filters

Use `filters` to limit which Markdown sources an instance processes. Patterns
are relative to the documentation directory. `include` admits matching
sources, and `exclude` removes matching sources after inclusion.

=== "`zensical.toml`"

    ``` toml
    [project.plugins."material/tags/public".filters]
    exclude = ["private/**"]
    ```

=== "`mkdocs.yml`"

    ``` yaml
    plugins:
      - material/tags/public:
          filters:
            exclude:
              - private/**
    ```

An empty `include` list accepts all sources. A source that an instance does
not accept does not contribute tags or consume listing directives for that
instance.

### Unsupported Material for MkDocs settings

Zensical does not create the `tags.json` export file. The `export` and
`export_file` settings are accepted for configuration compatibility but have no
effect. The `export_only` setting is not supported and causes a configuration
error.

The following deprecated settings are not supported and cause a configuration
error:

- `tags_file`
- `tags_extra_files`
- `tags_compare`
- `tags_compare_reverse`
- `tags_pages_compare`
- `tags_pages_compare_reverse`

Use a listing directive and the current sorting settings instead. Zensical also
does not execute arbitrary Python callables for slugification or sorting. Use
one of the supported strategies and compatible callable names documented on
this page.

## Page tags

### Add tags

Add a list to the page front matter:

``` yaml
---
tags:
  - Guide/Rust
  - Public
---
```

The default metadata property is `tags`. Use `tags_name_property` to read tags
from another property, and `tags_name_variable` to change the template
variable that receives the page's tag references.

=== "`zensical.toml`"

    ``` toml
    [project.plugins."material/tags"]
    tags_name_property = "labels"
    tags_name_variable = "labels"
    ```

=== "`mkdocs.yml`"

    ``` yaml
    plugins:
      - material/tags:
          tags_name_property: labels
          tags_name_variable: labels
    ```

Zensical renders tags on the page and exposes references to matching listings
in the configured `tags_name_variable` template variable. Set `tags = false` to
keep extracting tags and building listings without exposing page tag references
or rendering the page tag display.

### Tag display and search

Tags are rendered at the bottom of each page and can be used to filter search
results without further configuration.

To hide tags on an individual page, add `tags` to the page's `hide` property:

``` yaml
---
hide:
  - tags
---
```

### Tag icons and identifiers

Associate each tag with a unique identifier by adding the following to your
configuration:

=== "`zensical.toml`"

    ``` toml
    [project.extra.tags]
    <tag> = "<identifier>"
    ```

=== "`mkdocs.yml`"

    ``` yaml
    extra:
      tags:
        <tag>: <identifier>
    ```

The identifier can only include alphanumeric characters, dashes, and
underscores. For example, associate the `Compatibility` tag with the
`compat` identifier:

=== "`zensical.toml`"

    ``` toml
    [project.extra.tags]
    Compatibility = "compat"
    ```

=== "`mkdocs.yml`"

    ``` yaml
    extra:
      tags:
        Compatibility: compat
    ```

Identifiers can be reused between tags to assign groups of tags the same icon.
Tags without an explicitly associated identifier use the default tag icon.

Associate each identifier with an icon, including a [custom icon], by adding
the following to the `theme.icon.tag` configuration setting:

=== "`zensical.toml`"

    ``` toml
    [project.theme.icon.tag]
    default = "<icon>"
    <identifier> = "<icon>"
    ```

=== "`mkdocs.yml`"

    ``` yaml
    theme:
      icon:
        tag:
          default: <icon>
          <identifier>: <icon>
    ```

For example:

=== "`zensical.toml`"

    ``` toml
    [project.theme.icon.tag]
    default = "lucide/hash"
    html = "fontawesome/brands/html5"
    js = "fontawesome/brands/js"
    css = "fontawesome/brands/css3"

    [project.extra.tags]
    HTML5 = "html"
    JavaScript = "js"
    CSS = "css"
    ```

=== "`mkdocs.yml`"

    ``` yaml
    theme:
      icon:
        tag:
          default: lucide/hash
          html: fontawesome/brands/html5
          js: fontawesome/brands/js
          css: fontawesome/brands/css3
    extra:
      tags:
        HTML5: html
        JavaScript: js
        CSS: css
    ```

### Allowed tags

Set `tags_allowed` to reject tag names that are not in a predefined list. This
helps catch spelling errors in front matter.

``` toml
[project.plugins."material/tags"]
tags_allowed = ["Guide/Rust", "Guide/Python", "Public"]
```

The comparison is exact. Tag values that are not strings are converted to
their scalar string representation before validation.

### Hierarchical tags

Set `tags_hierarchy = true` to treat a separator in a tag name as a hierarchy.
The default separator is `/`:

``` toml
[project.plugins."material/tags"]
tags_hierarchy = true
tags_hierarchy_separator = "/"
```

`Guide/Rust` then has `Guide` as its parent and `Guide/Rust` as its leaf tag.
Each tag keeps its full name, and listings render the parent and child nodes as
a tree.

### Tag slugs

Tag slugs are used for listing anchors and tag reference URLs. The defaults are
equivalent to:

``` toml
[project.plugins."material/tags"]
tags_slugify = "pymdownx:lower"
tags_slugify_separator = "-"
tags_slugify_format = "tag:{slug}"
```

`tags_slugify_format` must contain the `{slug}` placeholder. The
`tags_slugify_separator` value is passed to the selected strategy. Zensical
supports these strategies:

- `pymdownx:lower`, the default. It preserves Unicode characters, converts
  text to lowercase, removes unsupported characters, and replaces spaces with
  the configured separator.
- `pymdownx:fold`, which uses Unicode case folding instead of lowercasing.
- `markdown:slugify`, which follows Python Markdown's ASCII-oriented slug
  behavior.

For compatibility with Material for MkDocs, `pymdownx.slugs.uslugify` is an
alias for `pymdownx:lower`. The `pymdownx.slugs.slugify` callable is also
supported with its `case` keyword set to `"lower"` or `"fold"`, and corresponds
to the matching `pymdownx` strategy. Its normalization must remain `NFC`, and
percent-encoding is not supported.

Zensical does not execute arbitrary custom Python slugification functions.

### Tag sorting

Use `tags_sort_by` and `tags_sort_reverse` to control the order of page tags.
The default strategy is `tag_name`, and the default order is ascending. The
supported tag strategies are:

- `tag_name`, which sorts by the tag name.
- `tag_name_casefold`, which sorts without case differences.

The strategies can be written as strings or with the compatible Material
callable names in `mkdocs.yml`:

``` yaml
plugins:
  - material/tags:
      tags_sort_by: tag_name_casefold
      tags_sort_reverse: true
```

## Tag listings

### Add a listing

Insert the default `material/tags` HTML comment where the listing should
appear:

``` html
<!-- material/tags -->
```

The listing can appear on any page. It selects matching pages from the whole
documentation by default. A page is not included in its own listing.

You can set the directive name with `listings_directive`:

``` toml
[project.plugins."material/tags"]
listings_directive = "$tags"
```

The corresponding comment is then `<!-- $tags -->`. Set `listings = false` to
stop replacing listing directives while retaining page tag extraction.

### Filter a listing

Inline listing configuration is written as YAML in the comment:

``` html
<!-- material/tags {
  scope: true,
  include: [Public],
  exclude: [Internal],
  shadow: false,
  toc: false
} -->
```

The supported listing settings are:

`scope`
: When `true`, include only pages below the directory that contains the
  listing.

`include`
: Include pages with the named tags. An empty value includes all tags and
  pages.

`exclude`
: Exclude a page when one of its tags or tag parents matches a named tag.

`shadow`
: Override the global `shadow` setting for this listing.

`toc`
: Override `listings_toc` for this listing. When `true`, listing tag nodes are
  added to the page table of contents.

`layout`
: Override the fragment layout used to render this listing.

### Reuse listing configurations

Define named configurations in `listings_map`, then refer to one by name in
the directive:

=== "`zensical.toml`"

    ``` toml
    [project.plugins."material/tags".listings_map.cards]
    include = ["Public"]
    layout = "cards"
    toc = false
    ```

=== "`mkdocs.yml`"

    ``` yaml
    plugins:
      - material/tags:
          listings_map:
            cards:
              include:
                - Public
              layout: cards
              toc: false
    ```

Use the name after the directive:

``` html
<!-- material/tags cards -->
```

Named configurations support `scope`, `shadow`, `layout`, `toc`, `include`,
and `exclude`. Inline settings can also override the selected listing
configuration.

### Listing layouts and sorting

Set `listings_layout` to choose the default fragment layout. The default is
`default`. A named listing can override this value with `layout`.

Use `listings_sort_by` and `listings_sort_reverse` to order pages inside a
listing. The default strategy is `item_title`. The other supported strategy is
`item_url`. Use `listings_tags_sort_by` and `listings_tags_sort_reverse` to
order tag nodes. The supported strategies are `tag_name` and
`tag_name_casefold`.

``` toml
[project.plugins."material/tags"]
listings_sort_by = "item_url"
listings_sort_reverse = true
listings_tags_sort_by = "tag_name_casefold"
listings_tags_sort_reverse = false
```

The corresponding Material callable names, including
`material.plugins.tags.item_title`, `material.plugins.tags.item_url`,
`material.plugins.tags.tag_name`, and
`material.plugins.tags.tag_name_casefold`, are accepted in `mkdocs.yml`.

### Listing table of contents

Listings add tag nodes to the page table of contents by default. Set
`listings_toc = false` globally or `toc: false` for one listing to disable
those entries.

## Shadow tags

Shadow tags are useful for tags that should be available in deploy previews but
not in a normal build. Set `shadow = true` to include them. The default is
`false` for builds and `true` for `serve`.

Use `shadow_on_serve` to change the serve default:

``` toml
[project.plugins."material/tags"]
shadow_on_serve = false
```

You can define shadow tags by exact name, prefix, or suffix:

``` toml
[project.plugins."material/tags"]
shadow_tags = ["Draft", "Internal"]
shadow_tags_prefix = "_"
shadow_tags_suffix = "Internal"
```

A page or listing can override the global setting with `shadow`. A shadowed
parent also hides its child tags when hierarchical tags are enabled.

## Multiple tag instances

Configure multiple named instances when different parts of a site need
separate source filters, metadata properties, or listing directives. Instances
are processed in configuration order and keep their settings isolated.

``` yaml
plugins:
  - material/tags/public:
      listings_directive: public/tags
      filters:
        exclude: [private/**]
  - material/tags/private:
      listings_directive: private/tags
      filters:
        include: [private/**]
      tags_name_property: labels
      tags_name_variable: labels
```

[custom icon]: ../setup/logo-and-icons.md#additional-icons
