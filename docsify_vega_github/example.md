# Hello crypto world !

## simple vega chart

```vega
https://raw.githubusercontent.com/vega/vega/master/docs/examples/bar-chart.vg.json
```

## first crypto chart

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
    "encoding": {"x": {"field": "0", "type": "temporal", "axis": {"title": "last day"}}},
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
          "y": {"field": "1", "type": "quantitative", "scale": {"zero": false}, "axis": {"title": "price in €"}}
        }
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

## advanced crypto chart

```vegalite

```

## distribution chart

```vegalite

```
