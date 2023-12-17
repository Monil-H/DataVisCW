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
londonGeoData =
    dataFromUrl "https://gicentre.github.io/data/geoTutorials/londonBoroughs.json"
        [ topojsonFeature "boroughs" ]

londonMostTravelData =
    dataFromUrl "https://raw.githubusercontent.com/Monil-H/DataVisCW/main/Data/MaxTravelMethodLondon.csv"


londonEnc =
    encoding
        << color
            [ mName "Observation"
            , mQuant
            ]
        << tooltips
            [ [  tName "Borough",tNominal ]
            , [ tName "TravelMethod" ,tNominal]
            , [ tName "Observation", tQuant]
            ]
        << opacity
            [ mCondition (prParamEmpty "mySelection")
                [ mNum 1.0 ]
                [ mNum 0.5 ]
            ]

cfg =
    configure
        << configuration (coView [ vicoStroke Nothing ])
        << configuration (coLegend [ lecoOrient loBottomRight, lecoOffset 0 ])

londonPS =
    params
        << param "mySelection"
            [ paBind
                (ipRadio
                    [ inName "Travel Method:"
                    , inOptions
                        [ "On foot"
                        , "Driving a car or van"
                        , "Bus, minibus or coach"
                        , "Underground, metro, light rail, tram"
                        ]
                    ]
                )
            , paSelect sePoint [ seFields [ "TravelMethod" ] ]]

proj =
    projection [ prType transverseMercator, prRotate 2 0 0 ]

```

```elm {interactive l v}
londonBoroughsChoropleth : Spec
londonBoroughsChoropleth =
    let
        trans =
            transform
                << lookup "id" (londonMostTravelData []) "Borough" (luFields [ "Observation","Borough","MethodCode","TravelMethod", "RegionCode" ])

    in
        toVegaLite [ width 600, height 450, cfg [], londonGeoData, trans [], proj, londonPS [], londonEnc [], geoshape [ maStroke "white" ] ]

```
