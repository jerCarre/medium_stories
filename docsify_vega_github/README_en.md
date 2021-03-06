---
generator: pandoc
tags:
- markdown
- vega
- vega-lite
- docsify
- github
title: How to display the price of a crypto on its Github page?
viewport: width=device-width, initial-scale=1.0, user-scalable=yes
---

# How to display the price of a crypto on its page Github page?

# How display the price of a crypto on its Github page?

Behind this racy title is an article on how to create a static site displaying a chart with dynamic data (ie. the the price of a crypto).

## What is Docsify?

The publication of static sites is becoming more and more fashionable, because implementation is often easy and the writing language is basic. basic. Several tools share the bill: [Jekyll](https://jekyllrb.com/), [Hugo](https://gohugo.io/)\... I chose another one: [Docsify](https://docsify.js.org). It has the advantage of to take markdown as input ( [I like](https://medium.com/@jerome.carre/au-rapport-chef-f186726a7de8) markdown ) [markdown](https://medium.com/@jerome.carre/au-rapport-chef-f186726a7de8) ) and especially to treat it as it is, it does not make any transformation into HTML.

Installing docsify is as simple as putting a file at the root of the site index.html file:

``` html
<!DOCTYPE html>
<html>
<head>
  <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <meta charset="UTF-8">
  <link rel="stylesheet" href="//cdn.jsdelivr.net/npm/docsify@4/themes/vue.css" />
</head>
<body>
  <div id="app"></div>
  <script>
    window.$docsify = {
      name: 'docsify vega',
      repo: 'https://github.com/jerCarre/docsify_vega',
    }
  </script>
  <script src="//cdn.jsdelivr.net/npm/docsify@4"></script>
</body>
</html>
```

and a content file README.md :

``` markdown
# README.md

Hello world !
```

> There is obviously a lot more that can be done with Docsify!

## Pages in Github

Github offers for free the Pages service to host a static site based on HTML, CSS, Javascript. It also provides a URL to access this site.

To use Pages, just: 1. create a public repo 2. go to `Settings->Pages`  3. choose the branch hosting your site 4. indicate the path to index.html of docsify.

In return Github will give you the access URL.

After a few seconds, your site is online !

![](https://jercarre.github.io/medium_stories/docsify_vega_github/empty.png)

So we will have the following files in our repo :

``` text
.
????????? /
    ????????? index.html
    ????????? README.md
```

## Vega-Lite forever

[Vega](https://vega.github.io/vega/) is a javascript library allowing to display diagrams (curves, scatterplots pie charts, maps, \...). It is based on a grammar in json format. [Vega-Lite](https://vega.github.io/vega-lite/) is a light version of Vega. Vega/Vega-Lite has the advantage of requiring only a few imports to work. Vega is intended to be embedded in other tools.

An overview of what you can draw with in [this previous article](https://medium.com/@jerome.carre/au-rapport-chef-f186726a7de8).

### Vega-Lite and docsify

Docsify allows you to display diagrams in Mermaid format. to integrate others you have to use plugins. No plugin being listed on [this page](https://docsify.js.org/#/awesome?id=plugins), so I developed one for Vega/Vega-Lite. Its use is described here https://jercarre.github.io/vega_docsify/#/

So we have our index.html file (with 4 more script lines at the end end):

``` html
<!DOCTYPE html>
<html>
<head>
  <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <meta charset="UTF-8">
  <link rel="stylesheet" href="//cdn.jsdelivr.net/npm/docsify@4/themes/vue.css" />
</head>
<body>
  <div id="app"></div>
  <script>
    window.$docsify = {
      name: 'docsify vega',
      repo: 'https://github.com/jerCarre/docsify_vega',
    }
  </script>
  <script src="//cdn.jsdelivr.net/npm/docsify@4"></script>
  
  <script src="//cdn.jsdelivr.net/npm/vega@5"></script>
  <script src="//cdn.jsdelivr.net/npm/vega-lite@5"></script>
  <script src="//cdn.jsdelivr.net/npm/vega-embed@6"></script>
  <script src="//cdn.jsdelivr.net/gh/jerCarre/vega_docsify@v1.1/lib/docsivega.js"></script>
  
</body>
</html>
```

and the README.md

```` {.sourceCode .markdown}
# Hello world !

```vega
https://raw.githubusercontent.com/vega/vega/master/docs/examples/bar-chart.vg.json
```
````

To obtain:

![](https://jercarre.github.io/medium_stories/docsify_vega_github/firstvega.png)

[DEMO](https://jercarre.github.io/medium_stories/#/docsify_vega_github/example?id=simple-vega-chart)

### A crypto in Vega-Lite

Vega/Vega-Lite also has the advantage of being able to read its data from an API. There are still some constraints: no header management and only in GET. After some research, the [coingecko](https://www.coingecko.com/en/api) API seems to be ideal. It allows to have the price of a crypto on the last X days, and does not days, and does not require a token to access.

> Insofar as the call towards this API is carried out by the navigator, it will be impossible browser, it will be impossible to hide the passage of a token in the the url.

To get the bitcoin price for the last 24 hours, you can call : https://api.coingecko.com/api/v3/coins/bitcoin/market_chart?vs_currency=eur&days=1

and have in response (as a shortcut) :

``` json
{"prices":[[1641137007567,41637.83411713587],[1641137244268,41579.072842123],[1641137476057,41578.04595490743],[1641137952530,41504.83017220277],[1641138178298,41502.722548713165],[1641138490632,41572.95862016224],[1641138725476,41664.34163465742],[1641139126440,41696.036160482596],[1641139324313,41706.24565959366],[1641139579229,41690.04405619627],[1641140038186,41750.43550340376],[1641140229278,41811.27689166205],[1641140453839,41735.801954558665],[1641140998579,41769.16264562593]]}
```

> The element \[0\] is the time (epoch), the \[1\] is the value in ???.

Just put it in a diagram:

```` {.sourceCode .markdown}
# Hello crypto world !

```vegalite
{
  "$schema": "https://vega.github.io/schema/vega-lite/v5.json",
  "description": "Bitcoin/eur last day evolution",
  "width": 500,
  "height": 300,
  "data": {
    "format": {"type": "json", "property": "prices"},
    "url": "https://api.coingecko.com/api/v3/coins/bitcoin/market_chart?vs_currency=eur&days=1"
  },
  "mark": "line",
  "encoding": {
    "x": {
      "field": "0", 
      "type": "temporal", 
      "axis": {"title": "last day"}
    },
    "y": {
      "field": "1",
      "type": "quantitative",
      "scale": {"zero": false},
      "axis": {"title": "price in ???"}
    }    
  }
}
```
````

![](https://jercarre.github.io/medium_stories/docsify_vega_github/firstcryptochart.png)

[DEMO](https://jercarre.github.io/medium_stories/#/docsify_vega_github/example?id=first-crypto-chart)

## and a little more \...

The diagram can be enriched with sliders that can be moved with the with the mouse on the curve. Also have the last value displayed on the right right side.

![](https://jercarre.github.io/medium_stories/docsify_vega_github/advancedcryptochart.png)

[DEMO](https://jercarre.github.io/medium_stories/#/docsify_vega_github/example?id=advanced-crypto-chart)

With these same data we can also display the price distribution :

![](https://jercarre.github.io/medium_stories/docsify_vega_github/distributionchart.png)

[DEMO](https://jercarre.github.io/medium_stories/#/docsify_vega_github/example?id=bitcoin-last-60-days-distribution-chart)

## In conclusion

As we have seen, we can deploy on Github in a few minutes a static site displaying dynamic data, and without any line of of HTML or javascript. We only focus on the content!

*Some useful links*

-   [The sources of this article and examples](https://jercarre.github.io/medium_stories/#/)
-   [Docsify](https://docsify.js.org)
-   [Vega-Lite](https://vega.github.io/vega-lite/)
-   [coingecko](https://www.coingecko.com/en/api)
-   A [Github action](https://github.com/philips-software/post-to-medium-action) to automatically publish on Medium
