# Hello crypto world !

## simple vega chart

```vega
https://raw.githubusercontent.com/vega/vega/master/docs/examples/bar-chart.vg.json
```

## First crypto chart

```vegalite
{
  "$schema": "https://vega.github.io/schema/vega-lite/v5.json",
  "description": "Bitcoin/eur last day evolution",
  "width": 800,
  "height": 600,
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
      "axis": {"title": "price in €"}
    }    
  }
}
```

## Advanced crypto chart

```vegalite
{
    "$schema": "https://vega.github.io/schema/vega-lite/v5.json",
    "description": "Bitcoin/eur last day evolution",
    "width": 800,
    "height": 600,
    "data": {
      "format": {"type": "json", "property": "prices"},
      "url": "https://api.coingecko.com/api/v3/coins/bitcoin/market_chart?vs_currency=eur&days=1"
    },  
    "layer": [
      {
        "layer": [
          {"mark": "line"},
          {
            "transform": [{"filter": {"param": "hover", "empty": false}}],
            "mark": "point"
          }
        ],
        "encoding": {
        "x": {"field": "0", "type": "temporal", "axis": {"title": "last day"}},
          "y": {"field": "1", "type": "quantitative", "scale": {"zero": false}, "axis": {"title": "price in €"}}
        }
      },
      {
      "encoding": {
      "x": {"aggregate": "max", "field": "0", "type": "temporal"},
      "y": {"aggregate": {"argmax": "0"}, "field": "1", "type": "quantitative"}
    },
    "layer": [{
      "mark": {"type": "circle"}
    }, {
      "mark": {"type": "text", "align": "left", "dx": 4},
      "encoding": {"text": {"field":"1", "type": "quantitative"}}
    }]
      },
      {
        "mark": "rule",
        "encoding": {
          "opacity": {
            "condition": {"value": 0.3, "param": "hover", "empty": false},
            "value": 0
          },
          "tooltip": [{"field": "1", "title": "price"}]
        },
        "params": [
          {
            "name": "hover",
            "select": {
              "type": "point",
              "fields": ["0"],
              "nearest": true,
              "on": "mouseover",
              "clear": "mouseout"
            }
          }
        ]
      }
    ]
  }
```

## Bitcoin last 60 days distribution chart

```vegalite
{
  "$schema": "https://vega.github.io/schema/vega-lite/v5.json",
  "width": 800,
  "height": 600,
  "data": {
    "url": "https://api.coingecko.com/api/v3/coins/bitcoin/market_chart?vs_currency=eur&days=60",
    "format": {"type": "json", "property": "prices"}
  },
  "transform": [
    { "calculate": "round(0.01*datum[1])/0.01", "as": "rounded" },
    {
      "aggregate": [{
       "op": "count",
       "field": "rounded",
       "as": "_count"
      }],
      "groupby": ["rounded"]
    }
  ],
  "mark": "area",
  "encoding": {
    "x": {"field": "rounded", "type": "quantitative", "title": "rounded price"},
    "y": {"field": "_count", "type": "quantitative", "title": false}
  }
}

```
