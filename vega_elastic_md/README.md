---
title: "Au rapport chef !!"
tags: 
  - markdown
  - vega
  - vega-lite
  - elasticsearch
---

## Genèse

Un peu comme dans la dernière émission culinaire à la mode on se dit parfois qu'en  mélangeant tel et tel ingrédient le résultat devrait être bon ! C'est d'une idée bizarre comme ça qu'est né cet article : "Si je mets des data `Elasticsearch` dans un diagramme `vega-lite`, dans un bloc de code `markdown` ça devrait être pros !"

![banner](https://jercarre.github.io/medium_stories/vega_elastic_md/banner.png)

L'idée a aussi été inspirée d'un outil : [Marktext](https://marktext.app/). C'est un éditeur wysiwyg de `markdown` qui sait interpréter en live/exporter le `vega/vega-lite` !

## Les bases

Un petit rappel des outils que l'on va mettre en œuvre :

[**Elasticsearch**](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html) : C'est un moteur d'indexation qui stocke les données en json et est requêtable via une API REST. Il est hautement scalable. Pour notre article on utilisera la version 7.6 en édition basic (gratuite) car les fonctionnalités *SQL* et *Transform* ne sont pas présentes dans la version Open Source.

[**Markdown**](https://en.wikipedia.org/wiki/Markdown) : C'est un langage léger pour rédiger des documents. Il se concentre sur le fond du document et non sur sa forme. Ce langage est très répandu, notamment dans les outils de gestion de configuration (github, gitlab ...). Le `markdown` ayant un format texte, il est complétement adapté à la gestion de conf (git).

[**Vega-lite**](https://vega.github.io/vega-lite/) : `vega` et sa version simplifiée `vega-lite` sont des grammaires de description de visualisations (diagrammes). Ce sont aussi des outils pour les rendre dans des environnements javascript. Le `markdown` permet notamment d'intégrer des blocs de code comme le `vega-lite`, ainsi ces diagrammes seront aussi gérés en conf. Un [éditeur en ligne](https://vega.github.io/editor/) permet de débugger le `vega/vega-lite`.

[**Marktext**](https://marktext.app/) : C'est un éditeur de `markdown` gratuit, opensource et multi-plateforme (c'est de l'electron) ! Il peut aussi faire des exports en PDF des diagrammes `vega-lite`. Dans cet article nous utiliserons la version 0.16.1.

## A l'origine ... les data

Pour avoir une situation de reporting réaliste, on doit avoir des données représentatives d'une activité métier.  `Elastic Stack` fournit justement des données de démonstration pour cela : d'e-commerce, de navigation aérienne, de trafic web. On choisira les données d'e-commerce. Chaque enregistrement trace l'achat d'un consommateur, avec : les produits achetés, la localisation de l'acheteur...

Une requête intéressante à afficher serait : le montant des ventes d'hier par pays.

 On prépare donc une belle requête `Elasticsearch` pour que les données arrivent pré-traitées dans `vega-lite` :

```json
POST _sql?format=csv
{
  "query": "SELECT SUM(taxful_total_price) AS total_price, geoip.country_iso_code AS country FROM kibana_sample_data_ecommerce GROUP bY geoip.country_iso_code ORDER BY count",
  "filter": { 
    "range": {
      "order_date": {~
          "gte" : "now-2d/d",
          "lte" : "now-1d/d"
} } } }
```

> On choisit d'utiliser l'API SQL car elle permet de simplifier l'écriture de la requête et de choisir le format de sortie (ex: CSV).

Il ne reste plus qu'à mettre cette URL et sa payload dans la partie data de `vega-lite` ... **Échec** !! `vega-lite` ne permet pas de passer une payload ni de choisir le verbe http ... on ne peut passer qu'une simple URL !

Pour dépasser cette limitation, nous avons deux possibilités :

1. on transforme les données dans `Elasticsearch`
2. on manipule les données dans `vega-lite`

Nous allons explorer les deux solutions.

### Transformation !!

Depuis la version 7.2, `Elasticsearch` propose l'API *[transform](https://www.elastic.co/guide/en/elasticsearch/reference/current/transform-apis.html)*. Elle permet de transformer, regrouper des données à la volée (ou en batch) et de les stocker dans un nouvel index. Nous n'aurons plus qu'à interroger ce nouvel index avec une simple requête et le tour est joué. Les *transform* peuvent être manipulés par API ou par un écran dans Kibana.

Dans notre cas, on va utiliser *transform* pour faire la somme des achats par pays par jour :

```json
PUT _transform/ecommerce_by_day
{
  "source": {
    "index": [ "kibana_sample_data_ecommerce" ],
    "query": { "match_all": {} }
  },
  "dest": { "index": "transform_kibana_sample_data_ecommerce" },
  "pivot": {
    "group_by": {
      "country_iso_code": {
        "terms": { "field": "geoip.country_iso_code" }
      },
      "order_date": {
        "date_histogram": { "field": "order_date", "calendar_interval": "1d" }
      }
    },
    "aggregations": {
      "taxful_price_sum": {
        "sum": { "field": "products.taxful_price" }
      }
    }
  }
}
```

Une fois créée, il faut lancer la transformation :

```json
POST _transform/ecommerce_by_day/_start
```

Et maintenant en une seule requête nous obtenons notre jeu de données pré-calculé :

```shell
GET transform_kibana_sample_data_ecommerce/_search?q="order_date:>now-2d+AND+order_date:<now-1d"
```

> Il faut juste préciser que l'on ne veut que les données d'hier.

### Manipulation

Si vous ne voulez pas utiliser les *transform* ou si votre version d’Elasticsearch ne le permet pas, il ne vous reste plus qu'à manipuler les données dans `vega-lite`.

Le principe est qu'on récupère les données brutes en entrée et ensuite on applique des transformations. Pour ne pas charger toutes les données on limitera la période de temps des données, ce qui se fait facilement en une seule requête (voir requête lancée sur la transformation).

On va commencer par voir comment déclarer des données dans `vega-lite`:

```json
"data": {
  "url": "http://elasticsearch:9200/kibana_sample_data_ecommerce/_search?q=order_date:>now-2d+AND+order_date:<now-1d",
  "format": {
    "type": "json",
    "property": "hits.hits"
}
```

> * une *url* contenant notre requête simple
> * un bloc *format* permettant de préciser le *type* des données (json pour nous)  et la racine des données : *hits.hits* dans les résultats `Elasticsearch`.

On fait l'hypothèse que les données remontées ne concernent que la journée d'hier.

On va donc pour simplement préciser que sur l'axe numérique (des montants) on veut faire une somme qui sera forcément corrélées à l'autre axe (des villes) :

```json
  "encoding": {
    "x": {
      "field": "_source.taxful_total_price",
      "aggregate": "sum",
      "type": "quantitative",
    },
    "y": {
      "field": "_source.geoip.country_iso_code",
      "type": "ordinal",
    }
  }
```

> les donnés sont disponibles dans le bloc *_source*

Ça parait plus simple mais l'agrégation est très simple ici. Dans la suite de l'article on privilégiera la solution à base de *transform* `Elasticsearch`. `vega/vega-lite` permet de faire des manipulations complexes des données (agrégations, bucket), mais veut-on vraiment charger notre document avec un code complexe ?

## Ooooh la belle courbe

Pour le type de rapport que l'on veut présenter (le montant des ventes d'hier par pays), on se tournerait naturellement vers une représentation en camembert ou donut (si vous êtes plus sucré que salé). Dans `vega-lite` cela s'appelle *arc*. Et ce n'est bizarrement disponible que depuis la version 4.9 (avril 2020) et ... `marktext` n'est qu'en version 4.7 => **Échec pas de représentation en camembert !**

Et bien on choisira une représentation en barres horizontales (*bar* en `vega-lite`).

```json
{
  "data": {
    "url": "http://elasticsearch:9200/transform_kibana_sample_data_ecommerce/_search?q=order_date:<now-1d+AND+order_date:>=now-2d",
    "format": {
      "type": "json",
      "property": "hits.hits"
    }
  },
  "mark": "bar",
  "encoding": {
    "x": {
      "field": "_source.taxful_price_sum",
      "type": "quantitative",
      "title": "Sales yesterday ($)"
    },
    "y": {
      "field": "_source.country_iso_code",
      "type": "ordinal",
      "title": "By country (code)"
    }
  }
}
```

TADAAAAM !!

![bar diagram](https://jercarre.github.io/medium_stories/vega_elastic_md/bar_diagram.png)

### Mais le plus beau reste à venir !!

Nos données contiennent le code du pays dont est issu l'achat. Pourquoi ne pas représenter tout cela sur une carte ?`vega-lite` peut manipuler des cartes au format *topojson*, je ne suis pas spécialiste, mais c'est globalement une description de la forme de chaque pays enrichie de métadonnées. Le format du code pays renvoyé par `Elasticsearch` est sur deux caractères, c'est la norme *ISO 3166-1 alpha-2*. Il ne reste plus qu'à trouver une carte du monde avec ce code pour identifier chaque pays. En voici [une](https://raw.githubusercontent.com/capta-journal/map/master/world/topojson/ne_110m_admin_0_countries.json).

Dans `vega-lite` on va donc déclarer les deux sources de données (la carte et les données) puis les joindre (lookup) en précisant la clef de jointure (world:properties.ISO_A2 → data:_source.country_iso_code). Puis on projetera le fond de carte vide et par dessus les pays portant des données avec un code couleur mettant en avant les plus gros acheteurs.

Voici le code `vega-lite` :

```json
{
  "$schema": "https://vega.github.io/schema/vega-lite/v4.json",
  "width": 900,
  "height": 500,
  "data": {
      "name": "world",
      "url": "https://raw.githubusercontent.com/capta-journal/map/master/world/topojson/ne_110m_admin_0_countries.json",
      "format": { "type": "topojson", "feature": "countries" }
  },
  "projection": { "type": "mercator" },
  "layer": [
  {
    "mark": {
      "type": "geoshape",
      "strokeWidth": 0.5,
      "stroke": "#bbb",
      "fill": "#e5e8d3"
    }
  },
  {
    "transform": [{
      "lookup": "properties.ISO_A2",
      "from": {
        "data": {
          "url": "http://elasticsearch:9200/transform_kibana_sample_data_ecommerce/_search?q=order_date:<now-1d+AND+order_date:>=now-2d",
          "format": { "type": "json", "property": "hits.hits" }
        },
        "key": "_source.country_iso_code",
        "fields": ["_source.taxful_price_sum"]
      }
    }],
    "mark": { "type": "geoshape" },
    "encoding": {
      "color": {
        "field": "_source.taxful_price_sum",
        "type": "quantitative",
        "title": "Sales yesterday ($)"
      }
    }
  }
  ]
}
```

Ooooooh

![wonderful world](https://jercarre.github.io/medium_stories/vega_elastic_md/world_data.png)

## Au rapport chef !!

Il ne reste plus qu'à mettre tout ça dans un document `markdown` avec `Marktext`. Cet outil nous permet d'ajouter rapidement certains blocs, il suffit de taper `@vega` et un bloc de code arrive. Une fois le code inséré, le rendu est automatique.

![marktext](https://jercarre.github.io/medium_stories/vega_elastic_md/marktext.png)

Sauf si votre boss est un peu geek, il n'appréciera pas votre `markdown` à sa juste valeur. Alors un export en PDF en choisissant le thème qui vous convient et le tour est joué !!

![pdf](https://jercarre.github.io/medium_stories/vega_elastic_md/pdf_marktext.png)

On a beaucoup parlé du `markdown` mais un autre langage permet d'intégrer des blocs de visualisations : `asciidoctor`. Lors de la transformation du document asciidoctor en pdf (ou autre) on peut préciser de rendre des diagrammes en image. Plus d'info sur [la doc officielle](https://asciidoctor.org/docs/asciidoctor-diagram/).

```asciidoc
=== Hic ensis iubeas nec

ore: hic mori numerant in, forsitan intus. Iaculum cepere et repulsae
maxima dominus recentes, capitum dabat titulos vitant votis, locus si

[vegalite]
----
{
  "data": {
    "url": "data_transform.json",
    "format": {
      "type": "json",
      "property": "hits.hits"
    }
  },
  "mark": "bar",
  "encoding": {
    "x": {
      "field": "_source.taxful_price_sum",
      "type": "quantitative",
      "title": "Sales($)"
    },
    "y": {
      "field": "_source.country_iso_code",
      "type": "ordinal",
      "title": "By country (code)"
    }
  }
}
----
```

Et le pdf ...

![pdf adoc](https://jercarre.github.io/medium_stories/vega_elastic_md/pdf_adoc.png)

## Conclusion

On a donc réussi à connecter une visualisation `vega-lite` avec une `elasticsearch` et de rendre ce diagramme dans du `markdown` puis un PDF. Cerise sur le gateau on a même réussi à projeter ces données sur un fond de carte, histoire d'avoir un rendu plus sexy ! On pourra imaginer de générer ces rapports via une chaîne CI/CD.

Mais tout ne s'est pas passé exactement comme on le souhaitait. Il faut donc se souvenir que :

* on ne peut avoir qu'une URL simple dans `vega-lite`, cela pose la question d'une authentification même par token.

* l'intégration de `vega-lite` dans `Marktext` (ou tout autre éditeur) est au bon vouloir des développeurs, on pourra donc être limité sur les fonctionnalités disponibles.

* la manipulation de données normées peut parfois s'avérer, paradoxalement, plus compliquée (ex: code pays)

## Sources

![gitbub](https://jercarre.github.io/medium_stories/GitHub-Mark-32px.png) : [repo github](https://github.com/jerCarre/elasticsearch_vegalite_markdown)
