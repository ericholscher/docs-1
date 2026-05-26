---
icon: lucide/braces
tags:
  - Extensions
---

# Macros

Zensical's Macros extension allows you to reuse and generate content. This helps
avoid repetitive and error-prone manual work. Once the extension is [configured
in your project], you can use [Jinja2 templating] in your Markdown content.

[configured in your project]: ../setup/extensions/macros.md
[Jinja2 templating]: https://jinja.palletsprojects.com/en/stable/

## Variables

Variables allow you to store values in one central location and reference them
throughout your documentation. Instead of updating a product name or version
number in dozens of files, you change it once in your configuration. You have
flexibility in how you define your variables.

__Configuration variables__ are defined in your configuration file under the `extra`
section. These are perfect for project-wide data like `version`,
`support_email`, or `company_name`.

=== "`zensical.toml`"
    ```toml
    [project.extra]
    product = "Zensical"
    version = "0.0.41"
    ```

=== "`mkdocs.yml`"
    ```yaml
    extra:
      product: "Zensical"
      version: "0.0.41"
    ```

__Page-level variables__ are defined directly inside a Markdown page. This is
useful for information specific to a single page.

```jinja
{% set feature = "Macros" %}
{% set since_version = "0.0.39" %}
```

### Referencing variables

Simply wrap the variable name in double curly braces:

```jinja
Support for {{ feature }} in {{ extra.product }} was introduced
in {{ since_version }}.
```

!!! tip "Simplified variable names"
    Configuration variables that appear in `extra` are also available without
    the `etxra.` prefix, so in the above, `{{ product }}` would have worked as
    well. TODO: WHEN? CLASHES?

!!! tip "Escaping curly braces"
    If you need to use curly braces as literal text anywhere in your content,
    you need to escape the first two, as in `\{\{text in curly braces}}`.


### Listing available variables

The setup page contains a [list of variables] that Zensical makes available.

[list of variables]: ../setup/extensions/macros.md#built-in-template-variables

If you ever need to verify what variables are currently available to your pages,
you can add `{{ macros_info() }}` to any page. This will output a list of your
configured variables and environment data in your browser, helping you debug
your documentation setup.

## Including external Markdown files

If you have repetitive content such as legal disclaimers, standard setup
instructions, or shared glossaries, you can save that content as a separate file
and include it wherever needed. This is an alternative to using the [snippet
extension].

[snippet extension]: ../setup/extensions/python-markdown-extensions/#snippets

After creating your snippet file (e.g., `snippets/disclaimer.md`), use the
`include` command in your target Markdown file:

```jinja
{{ include("snippets/disclaimer.md") }}
```

This ensures that if your disclaimer changes, you only have to edit the source file once for it to update across the entire site.

!!! tip "Macros in includes are processed"
    If a snippet included with `include` contains macros, these will be
    evaluated in the context of the page being processed. So, you can
    reference variables in your snippets or even include other snippets.

## Including CSV data

Tabular data can be read from a range of formats and rendered to a Markdown
table. Note that [configuration] beyond the basic configuration of the macros
extension is needed for this. Once set up, data files can be read by specifying
a path relative to the project root:

```jinja
{{ read_csv("data/team.csv") }}
```

[configuration]: ../setup/extensions/macros.md#reading-tabular-data

!!! tip "Complex tables"
    Note that the Markdown table syntax allows only for relative simple tables.
    For more complex use cases, see the [macros extensions setup page]. It
    is possible, for example, to write a custom filter that outputs HTML
    tables that can, effectively, have any shape.

 [macros extensions setup page]: ../setup/extensions/macros.md

## Conditional content

Conditional logic allows you to show or hide sections of text based on specific
criteria (e.g., hiding a feature description if the reader is viewing
documentation for a different edition).

Using the `if` statement, you can create pages that adapt to different user
needs:

```jinja
{% if extra.edition == 'pro' %}
### Advanced Analytics
Access your data insights via the dashboard...
{% endif %}
```

If the `extra.edition` variable is set to `pro`, the content appears. If not, it
is ignored, allowing you to maintain one single page for multiple audiences.

## Content from lists

If you have variables that contain lists then you can use loops to produce
content for each of the list items:

```jinja
{% for author in authors %}
* {{ author }}
{% endfor %}
```

Assuming the list of authors is `["Moe", "Larry", "Curly"]`, then the loop would
produce:

```markdown
* Moe
* Larry
* Curly
```



