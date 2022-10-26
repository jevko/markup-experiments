# Jevko markup experiments

A collection of experiments with [Jevko](https://jevko.org) and text markup. Translating Jevko to HTML/XML.

## parse

`parse` is a parser for the following ABNF grammar:

```fs
Value = Subvalues Suffix
Subvalue = Prefix "[" Value "]"

Subvalues = *Subvalue
Suffix = *Char
Prefix = *Char

Char = Escape / %x0-5a / %x5c / %x5e-5f / %x61-10ffff
Escape = "`" ("`" / "[" / "]")
```

which is equivalent to the [Jevko](https://jevko.org) grammar.

The shape of the trees returned by `parse` fairly closely matches the grammar, e.g.:

```json
{
  "subvalues": [
    {
      "prefix": "key ",
      "value": {
        "subvalues": [],
        "suffix": "value"
      }
    }
  ],
  "suffix": ""
}
```

is the syntax tree of the string:

```
key [value]
```

## astToHtml

`astToHtml` converts a parse tree returned by `parse` into HTML/XML string, effectively translating a compact Jevko-based encoding of XML to XML itself. For example the following Jevko string:

```
[html][
  [head][
    [meta /]
  ]
  [body][
    [p title[explanation] disabled][
      [b][click] on this [a href[#]][link][br/]
    ]
  ]
]
```

can be translated to the following XML string:

```html
<html>
  <head>
    <meta />
  </head>
  <body>
    <p title="explanation" disabled>
      <b>click</b> on this <a href="#">link</a><br/>
    </p>
  </body>
</html>
```

## astToXml

```
<html><
  <head><
    <meta />
  >
  <body><
    <p title<explanation> disabled><
      <b><click> on this <a href<#>><link><br/>
    >
  >
```

## astToXml2

```
[html/
  [head/
    [meta]
  ]
  [body/
    [p title[explanation] disabled/
      [b/click] on this [a href[#]/link][br]
    ]
  ]
]
```

## astToXml3

XML subset with no attributes

```
svg[width[391] height[391] viewBox[-70.5 -70.5 391 391] xmlns[http://www.w3.org/2000/svg] xmlns:xlink[http://www.w3.org/1999/xlink]
rect[fill[#fff] stroke[#000] x[-70] y[-70] width[390] height[390]]
g[opacity[0.8]
	rect[x[25] y[25] width[200] height[200] fill[lime] stroke-width[4] stroke[pink]]
	circle[cx[125] cy[125] r[75] fill[orange]]
	polyline[points[50,150 50,200 200,200 200,100] stroke[red] stroke-width[4] fill[none]]
	line[x1[50] y1[50] x2[200] y2[200] stroke[blue] stroke-width[4]]
]
]
```

## astToXml4

like astToXml3, but supports attributes which are separated from content by `/`

```
svg[width[391] height[391] viewBox[-70.5 -70.5 391 391] xmlns[http://www.w3.org/2000/svg] xmlns:xlink[http://www.w3.org/1999/xlink]/
rect[fill[#fff] stroke[#000] x[-70] y[-70] width[390] height[390]]
g[opacity[0.8]/
	rect[x[25] y[25] width[200] height[200] fill[lime] stroke-width[4] stroke[pink]]
	circle[cx[125] cy[125] r[75] fill[orange]]
	polyline[points[50,150 50,200 200,200 200,100] stroke[red] stroke-width[4] fill[none]]
	line[x1[50] y1[50] x2[200] y2[200] stroke[blue] stroke-width[4]]
]
]
```

## astToXml5

attributes are marked with `=`

elements with empty tags resolve to text nodes

```
phyloxml[xmlns:xsi=[http://www.w3.org/2001/XMLSchema-instance] xmlns=[http://www.phyloxml.org] xsi:schemaLocation=[http://www.phyloxml.org http://www.phyloxml.org/1.10/phyloxml.xsd]
  phylogeny[rooted=[true]
    name[Alcohol dehydrogenases]
    description[contains examples of commonly used elements]
    clade[
      events[
        speciations[1]
      ]
      clade[
        taxonomy[
          id[provider=[ncbi]6645]
          scientific_name[Octopus vulgaris]
        ]
        sequence[
          accession[source=[UniProtKB]P81431]
          name[Alcohol dehydrogenase class-3]
        ]
      ]
      clade[
        confidence[type=[bootstrap]100]
        events[
          speciations[1]
        ]
        clade[
          taxonomy[
            id[provider=[ncbi]1423]
            scientific_name[Bacillus subtilis]
          ]
          sequence[
            accession[source=[UniProtKB]P71017]
            name[Alcohol dehydrogenase]
          ]
        ]
        clade[
          taxonomy[
            id[provider=[ncbi]562]
            scientific_name[Escherichia coli]
          ]
          sequence[
            accession[source=[UniProtKB]Q46856]
            name[Alcohol dehydrogenase]
          ]
        ]
      ]
    ]
  ]
]
```

## astToXml6

like astToXml5, but attributes are of the form `tag [[attributes] children]`, i.e. they come as the first child with empty tag. This means that to have a tag with no attributes and a text node as the first child one must write `tag [[][text]]`

```
phyloxml[[xmlns:xsi[http://www.w3.org/2001/XMLSchema-instance] xmlns[http://www.phyloxml.org] xsi:schemaLocation[http://www.phyloxml.org http://www.phyloxml.org/1.10/phyloxml.xsd]]
  phylogeny[[rooted[true]]
    name[Alcohol dehydrogenases]
    description[contains examples of commonly used elements]
    clade[
      events[
        speciations[1]
      ]
      clade[
        taxonomy[
          id[[provider[ncbi]]6645]
          scientific_name[Octopus vulgaris]
        ]
        sequence[
          accession[[source[UniProtKB]]P81431]
          name[Alcohol dehydrogenase class-3]
        ]
      ]
      clade[
        confidence[[type[bootstrap]]100]
        events[
          speciations[1]
        ]
        clade[
          taxonomy[
            id[[provider[ncbi]]1423]
            scientific_name[Bacillus subtilis]
          ]
          sequence[
            accession[[source[UniProtKB]]P71017]
            name[Alcohol dehydrogenase]
          ]
        ]
        clade[
          taxonomy[
            id[[provider[ncbi]]562]
            scientific_name[Escherichia coli]
          ]
          sequence[
            accession[[source[UniProtKB]]Q46856]
            name[Alcohol dehydrogenase]
          ]
        ]
      ]
    ]
  ]
]
```

## astToXml7

Like astToXml6, except attributes come immediately after the tag, separated by a space. Out of all of the above this encoding is the most compact for data interchange.

```
svg width[391] height[391] viewBox[-70.5 -70.5 391 391] xmlns[http://www.w3.org/2000/svg] xmlns:xlink[http://www.w3.org/1999/xlink][
  rect fill[#fff] stroke[#000] x[-70] y[-70] width[390] height[390][]
  g opacity[0.8][
    rect x[25] y[25] width[200] height[200] fill[lime] stroke-width[4] stroke[pink][]
    circle cx[125] cy[125] r[75] fill[orange][]
    polyline points[50,150 50,200 200,200 200,100] stroke[red] stroke-width[4] fill[none][]
    line x1[50] y1[50] x2[200] y2[200] stroke[blue] stroke-width[4][]
  ]
]
```

## astToXml8

Like astToXml7, except that a tag without content may omit empty brackets if it is followed by a tag with attributes.

```
svg width[391] height[391] viewBox[-70.5 -70.5 391 391] xmlns[http://www.w3.org/2000/svg] xmlns:xlink[http://www.w3.org/1999/xlink][
  rect fill[#fff] stroke[#000] x[-70] y[-70] width[390] height[390]
  g opacity[0.8][
    rect x[25] y[25] width[200] height[200] fill[lime] stroke-width[4] stroke[pink]
    circle cx[125] cy[125] r[75] fill[orange]
    polyline points[50,150 50,200 200,200 200,100] stroke[red] stroke-width[4] fill[none]
    line x1[50] y1[50] x2[200] y2[200] stroke[blue] stroke-width[4]
  ]
]
```

In this encoding shorthand attributes without values must be placed immediately after the tag, and if they any are specified, they must be followed by a non-shorthand attribute i.e.:

```
elem attr attr2 attr3[][content]
```

## astToHtmlTable

Converts a little DSL for HTML tables into HTML.

For example this:

```
#[[decimal][hexadecimal][decimal-encoded hexadecimal]]
 [[1]      [1]          [1]                          ]
 [[12]     [c]          [u[03]]                      ]
 [[57]     [20]         [[2]u[00]]                   ]
 [[128]    [80]         [[8]u[00]]                   ]
 [[1000]   [3e8]        [[3]u[05]8]                  ]
 [[5598]   [15de]       [[15]u[04]u[05]]             ]
```

turns into something like this:

```
<table>
  <tr><th>decimal</th><th>hexadecimal</th><th>decimal-encoded hexadecimal</th></tr>
  <tr><td>1</td>      <td>1</td>          <td>1</td>                          </tr>
  <tr><td>12</td>     <td>c</td>          <td><u>03</u></td>                  </tr>
  <tr><td>57</td>     <td>20</td>         <td>2<u>00</u></td>                 </tr>
  <tr><td>128</td>    <td>80</td>         <td>8<u>00</u></td>                 </tr>
  <tr><td>1000</td>   <td>3e8</td>        <td>3<u>05</u>8</td>                </tr>
  <tr><td>5598</td>   <td>15de</td>       <td>15<u>04</u><u>05</u></td>       </tr>
</table>
```
