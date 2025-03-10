HTML back-end
=============

```@meta
CurrentModule = PrettyTables
DocTestSetup = quote
    using PrettyTables
end
```

```@raw html
<script language="javascript" type="text/javascript">
 function resizeIframe(obj)
 {
   obj.style.height = obj.contentWindow.document.body.scrollHeight + 10 + 'px';
   obj.style.width = obj.contentWindow.document.body.scrollWidth + 100 + 'px';
 }
</script>
```

The following options are available when the HTML backend is used. Those can be
passed as keywords when calling the function [`pretty_table`](@ref):

- `allow_html_in_cells::Bool`: By default, special characters like `<`, `>`,
    `"`, etc. are replaced in HTML backend to generate valid code. However, this
    algorithm blocks the usage of HTML code inside of the cells. If this keyword
    is `true`, then the escape algorithm **will not** be applied, allowing HTML
    code inside all the cells. In this case, the user must ensure that the
    output code is valid. If only few cells have HTML code, wrap then in a
    [`HtmlCell`](@ref) object instead. (**Default** = `false`)
- `continuation_row_alignment::Symbol`: A symbol that defines the alignment of
    the cells in the continuation row. This row is printed if the table is
    vertically cropped. (**Default** = `:r`)
- `highlighters::Union{HtmlHighlighter, Tuple}`: An instance of
    [`HtmlHighlighter`](@ref) or a tuple with a list of HTML highlighters (see
    the section [HTML highlighters](@ref)).
- `linebreaks::Bool`: If `true`, then `\\n` will be replaced by `<br>`.
    (**Default** = `false`)
- `maximum_columns_width::String`: A string with the maximum width of each
    columns. This string must contain a size that is valid in HTML. If it is not
    empty, then each cell will have the following style:
    - `"max-width": <value of maximum_column_width>`
    - `"overflow": "hidden"`
    - `"text-overflow": "ellipsis"`
    - `"white-space": "nowrap"`
    If it is empty, no additional style is applied. (**Default** = "")
- `standalone::Bool`: If `true`, then a complete HTML page will be generated.
    Otherwise, only the content between the tags `<table>` and `</table>` will
    be printed (with the tags included). (**Default** = `false`)
- `vcrop_mode::Symbol`: This variable defines the vertical crop behavior. If it
    is `:bottom`, then the data, if required, will be cropped in the bottom. On
    the other hand, if it is `:middle`, then the data will be cropped in the
    middle if necessary. (**Default** = `:bottom`)
- `table_div_class::String`: The class name for the table `div`. It is only used
    if `wrap_table_in_div` is `true`. (**Default** = "")
- `table_class::String`: The class name for the table. (**Default** = "")
- `table_style::Dict{String, String}`: A dictionary containing the CSS
    properties and their values to be added to the table `style`.
    (**Default** = `Dict{String, String}()`)
- `tf::HtmlTableFormat`: An instance of the structure [`HtmlTableFormat`](@ref)
    that defines the general format of the HTML table.
- `top_left_str::String`: String to be printed at the left position of the top
    bar. (**Default** = "")
- `top_left_str_decoration::HtmlDecoration`: Decoration used to print the
    top-left string (see `top_left_str`). (**Default** = `HtmlDecoration()`)
- `top_right_str::String`: String to be printed at the right position of the
    top bar. Notice that this string will be replaced with the omitted cell
    summary if it must be displayed. (**Default** = "")
- `top_right_str_decoration::HtmlDecoration`: Decoration used to print the
    top-right string (see `top_right_str`). (**Default** = `HtmlDecoration()`)
- `wrap_table_in_div::Bool`: If `true`, the table will be wrapped in a `div`.
    (**Default**: `false`)

## HTML highlighters

A set of highlighters can be passed as a `Tuple` to the `highlighters` keyword.
Each highlighter is an instance of the structure [`HtmlHighlighter`](@ref). It
contains the following two public fields:

- `f::Function`: Function with the signature `f(data, i, j)` in which should
    return `true` if the element `(i,j)` in `data` must be highlighted, or
    `false` otherwise.
- `fd::Function`: Function with the signature `f(h, data, i, j)` in which `h` is
    the highlighter. This function must return the `HtmlDecoration` to be
    applied to the cell that must be highlighted.

The function `f` has the following signature:

    f(data, i, j)

in which `data` is a reference to the data that is being printed, and `i` and
`j` are the element coordinates that are being tested. If this function returns
`true`, then the highlight style will be applied to the `(i, j)` element.
Otherwise, the default style will be used.

If the function `f` returns true, then the function `fd(h, data, i, j)` will be
called and must return an element of type [`HtmlDecoration`](@ref) that contains
the decoration to be applied to the cell.

A HTML highlighter can be constructed using two helpers:

```julia
HtmlHighlighter(f::Function, decoration::HtmlDecoration)
HtmlHighlighter(f::Function, fd::Function)
```

The first will apply a fixed decoration to the highlighted cell specified in
`decoration` whereas the second let the user select the desired decoration by
specifying the function `fd`.

!!! info
    If only a single highlighter is wanted, then it can be passed directly to
    the keyword `highlighter` without being inside a `Tuple`.

!!! note
    If multiple highlighters are valid for the element `(i, j)`, then the
    applied style will be equal to the first match considering the order in the
    tuple `highlighters`.

!!! note
    If the highlighters are used together with [Formatters](@ref), then the
    change in the format **will not** affect the parameter `data` passed to the
    highlighter function `f`. It will always receive the original, unformatted
    value.

There are a set of pre-defined highlighters (with names `hl_*`) to make the
usage simpler. They are defined in the file
`./src/backends/html/predefined_highlighters.jl`.

```julia
julia> t = 0:1:20;

julia> data = hcat(t, ones(length(t)) * 1, 1 * t, 0.5 .* t.^2);

julia> header = (["Time", "Acceleration", "Velocity", "Distance"],
                 [ "[s]",       "[m/s²]",    "[m/s]",      "[m]"]);

julia> hl_v = HtmlHighlighter((data, i, j) -> (j == 3) && data[i, 3] > 9, HtmlDecoration(color = "blue", font_weight = "bold"));

julia> hl_p = HtmlHighlighter((data, i, j) -> (j == 4) && data[i, 4] > 10, HtmlDecoration(color = "red"));

julia> hl_e = HtmlHighlighter((data, i, j) -> data[i, 1] == 10, HtmlDecoration(background = "black", color = "white"))

julia> pretty_table(data; backend = Val(:html), header = header, highlighters = (hl_e, hl_p, hl_v), standalone = true)
```

```@raw html
<iframe src="html_highlighters_example.html" frameborder="0" scrolling="no" onload="javascript:resizeIframe(this)">
  <p>Your browser does not support iframes. Click <a href="html_highlighters_example.html>here</a> to see the table.</p>
</iframe>
```

## HTML table formats

The following table formats are available when using the html back-end:

`tf_html_default` (**Default**)

```@raw html
<iframe src="html_format_default.html" frameborder="0" scrolling="no" onload="javascript:resizeIframe(this)">
  <p>Your browser does not support iframes. Click <a href="html_format_default.html>here</a> to see the table.</p>
</iframe>
```

`tf_html_dark`

```@raw html
<iframe src="html_format_dark.html" frameborder="0" scrolling="no" onload="javascript:resizeIframe(this)">
  <p>Your browser does not support iframes. Click <a href="html_format_dark.html>here</a> to see the table.</p>
</iframe>
```

`tf_html_minimalist`

```@raw html
<iframe src="html_format_minimalist.html" frameborder="0" scrolling="no" onload="javascript:resizeIframe(this)">
  <p>Your browser does not support iframes. Click <a href="html_format_minimalist.html>here</a> to see the table.</p>
</iframe>
```

`tf_html_matrix`

```@raw html
<iframe src="html_format_matrix.html" frameborder="0" scrolling="no" onload="javascript:resizeIframe(this)">
  <p>Your browser does not support iframes. Click <a href="html_format_matrix.html>here</a> to see the table.</p>
</iframe>
```

!!! info
    In this case, the table format `html_matrix` was printed with the option
    `noheader = true`.

`tf_html_simple`

```@raw html
<iframe src="html_format_simple.html" frameborder="0" scrolling="no" onload="javascript:resizeIframe(this)">
  <p>Your browser does not support iframes. Click <a href="html_format_simple.html>here</a> to see the table.</p>
</iframe>
```
