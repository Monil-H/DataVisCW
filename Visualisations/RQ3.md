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

```elm {l=hidden}

wfhEnglandGeoData =
    dataFromUrl "https://gicentre.github.io/data/census21/ewLAs.json"
        [ topojsonFeature "las" ]

workFromHomeData =
    dataFromUrl "https://raw.githubusercontent.com/Monil-H/DataVisCW/main/Data/WFHLowerTierLocalAuthoritesExtra.csv"

proj =
            projection [ prType transverseMercator, prRotate 2 0 0 ]

workFromHomeTrans =
    transform
        << lookup "properties.code" (workFromHomeData []) "AreaCode" (luFields [ "AreaCode" ,"Observation","AreaName","MethodCode","TravelMethod","PercentOfArea" ])
cfg =
    configure
        << configuration (coView [ vicoStroke Nothing ])
        << configuration (coLegend [ lecoOrient loBottomRight, lecoOffset 0 ])

```

```elm {interactive l v highlight=15}
wfhEnglandChoroplethObs : Spec
wfhEnglandChoroplethObs =
    let
        enc =
            encoding
                << color
                    [ mName "Observation"
                    , mQuant
                    , mTitle "Population that work from Home"
                    ]
                << tooltips
                    [ [  tName "AreaName",tNominal,tTitle "Region"  ]
                    , [ tName "Observation", tQuant]
                    , [ tName "PercentOfArea", tQuant]
                    ]


    in
    toVegaLite [width 600, height 750, cfg [], wfhEnglandGeoData, workFromHomeTrans [], proj, enc [], geoshape [maStroke "black"]]
```

```elm {interactive l v highlight=15}
wfhEnglandChoroplethPer : Spec
wfhEnglandChoroplethPer =
    let
        enc =
            encoding
                << color
                    [ mName "PercentOfArea"
                    , mQuant
                    , mTitle "Percent of area that work from home"
                    ]
                << tooltips
                    [ [  tName "AreaName",tNominal,tTitle "Region"  ]
                    , [ tName "Observation", tQuant]
                    , [ tName "PercentOfArea", tQuant]
                    ]


    in
    toVegaLite [width 600, height 750, cfg [], wfhEnglandGeoData, workFromHomeTrans [], proj, enc [], geoshape [maStroke "black"]]
```
