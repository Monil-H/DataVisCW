---
id: litvis

elm:
  dependencies:
    gicentre/elm-vegalite: latest
    gicentre/tidy: latest
---

```elm {l=hidden}
import Tidy exposing (..)
import VegaLite exposing (..)
```

```elm {interactive l v highlight=15}
londonChoropleth2 : Spec
londonChoropleth2 =
    let
        geoData =
            dataFromUrl "https://gicentre.github.io/data/census21/ewLAs.json"
                [ topojsonFeature "las" ]

        londonTravelData =
            dataFromUrl "https://raw.githubusercontent.com/Monil-H/DataVisCW/main/Data/WFHLowerTierLocalAuthoritesExtra.csv"


        trans =
            transform
                << lookup "properties.label" (londonTravelData []) "AreaName" (luFields [ "Observation","AreaName","MethodCode","TravelMethod","PercentOfArea" ])

        proj =
            projection [ prType transverseMercator, prRotate 2 0 0 ]

        enc =
            encoding
                << color
                    [ mName "Observation"
                    , mQuant
                    , mTitle "Population that work from Home"
                    ]
                << tooltips
                    [ [  tName "properties.label",tNominal,tTitle "Region"  ]
                    , [ tName "Observation", tQuant]
                    ]

        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coLegend [ lecoOrient loBottomRight, lecoOffset 0 ])
    in
    toVegaLite
        [ width 600
        , height 450
        , cfg []
        , geoData
        , trans []
        , proj
        , enc []
        , geoshape [ maStroke "white" ]
        ]
```
