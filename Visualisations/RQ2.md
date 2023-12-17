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
londonChoropleth : Spec
londonChoropleth =
    let
        geoData =
            dataFromUrl "https://gicentre.github.io/data/geoTutorials/londonBoroughs.json"
                [ topojsonFeature "boroughs" ]

        londonTravelData =
            dataFromUrl "https://raw.githubusercontent.com/Monil-H/DataVisCW/main/Data/LondonBoroughTravelMethods.csv"

        londonTravelData2 =
            dataFromUrl "https://raw.githubusercontent.com/Monil-H/DataVisCW/main/Data/MaxTravelMethodLondon.csv"


        trans =
            transform
                << lookup "id" (londonTravelData2 []) "Borough" (luFields [ "Observation","Borough","MethodCode","TravelMethod" ])
                --<< filter (fiExpr "datum.Borough == 'Barnet'")


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
                    [ [  tName "Borough",tNominal ]
                    , [ tName "TravelMethod" ,tNominal]
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
        , geoshape [ maStroke "yellow" ]
        ]
```
