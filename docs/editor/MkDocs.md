# MkDocs

- [配置](https://mkdocs.readthedocs.io/en/0.10/user-guide/configuration/)


```
mkdocs serve --dev-addr=0.0.0.0:80  # Run on port 80, accessible over the local network.
```

- [第三方主题](https://github.com/mkdocs/mkdocs/wiki/MkDocs-Themes)

## Material for MkDocs

[Getting started](https://squidfunk.github.io/mkdocs-material/getting-started/)

``` shell
pip install mkdocs && mkdocs --version
pip install mkdocs-material
```

mkdocs.yml中添加:

```
theme:
  name: material
```

[TOC扩展](https://squidfunk.github.io/mkdocs-material/extensions/permalinks/)

```
markdown_extensions:
  - toc:
    permalink: true
```

!!! info "信息"
    编程语言高亮: [highlight.js - Syntax highlighting for the Web](https://highlightjs.org/)



!!! warning "警告"
    提示和警告文本: [Admonition](https://squidfunk.github.io/mkdocs-material/extensions/admonition/)

!!! info
    [PyMdown Extensions](https://squidfunk.github.io/mkdocs-material/extensions/pymdown/): a collection of Markdown extensions that add some great missing features to the standard Markdown library.

!!! info
    [Arithmatex](https://squidfunk.github.io/mkdocs-material/extensions/pymdown/): an extension that preserves LaTeX math equations during the Markdown conversion process so that they can be used with libraries like MathJax.

$$
E(\mathbf{v}, \mathbf{h}) = -\sum_{i,j}w_{ij}v_i h_j - \sum_i b_i v_i - \sum_j c_j h_j
$$

\[3 < 4\]

\begin{align}
  p(v_i=1|\mathbf{h}) & = \sigma\left(\sum_j w_{ij}h_j + b_i\right) \\
  p(h_j=1|\mathbf{v}) & = \sigma\left(\sum_i w_{ij}v_i + c_j\right)
\end{align}


脚注: [^1]

[^1]: https://squidfunk.github.io/mkdocs-material/extensions/footnotes/


### Mermaid

[引入mermaid diagrams](https://github.com/squidfunk/mkdocs-material/issues/693):

```
markdown_extensions:
  - pymdownx.superfences:
      custom_fences:
        - name: mermaid
          class: mermaid
          format: !!python/name:pymdownx.superfences.fence_div_format

extra_javascript:
  - https://cdn.jsdelivr.net/npm/mermaid/dist/mermaid.min.js
```

```mermaid
graph TD;
    A-->B;
    A-->C;
    B-->D;
    C-->D;
```

### Graphviz

https://github.com/sprin/markdown-inline-graphviz/issues/7

```
pip install markdown-inline-graphviz-extension


<div>
{% dot attack_plan.svg
    digraph G {
        rankdir=LR
        Earth [peripheries=2]
        Mars
        Earth -> Mars
    }
%}
</div>
```
