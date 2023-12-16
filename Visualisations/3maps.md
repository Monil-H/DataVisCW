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
            dataFromUrl "https://raw.githubusercontent.com/Monil-H/DataVisCW/main/Data/WFHLondonBorough.csv"


        trans =
            transform
                --<< filter (fiExpr "datum.MethodCode /= 12")
                << lookup "id" (londonTravelData []) "Borough" (luFields [ "Observation","Borough","MethodCode","TravelMethod" ])
               --  << filter (fiEqual "MethodCode" (num 1))
                -- << filter (fiExpr "datum.MethodCode == 2")
                -- << filter (fiExpr "datum.MethodCode < 2 && datum.Borough == 'Brent'")


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

```elm {interactive l v highlight=15}
londonChoropleth2 : Spec
londonChoropleth2 =
    let
        geoData =
            dataFromUrl "https://gicentre.github.io/data/census21/ewLAs.json"
                [ topojsonFeature "las" ]

        londonTravelData =
            dataFromUrl "https://raw.githubusercontent.com/Monil-H/DataVisCW/main/Data/WFHLowerTierLocalAuthorites.csv"


        trans =
            transform
                << lookup "properties.label" (londonTravelData []) "AreaName" (luFields [ "Observation","Borough","MethodCode","TravelMethod" ])

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

```elm {interactive l v highlight=15}
londonChoropleth3 : Spec
londonChoropleth3 =
    let
        geoData =
            dataFromUrl "https://raw.githubusercontent.com/gicentre/data/main/census21/ewRegions.json"
                [ topojsonFeature "regions" ]

        londonTravelData =
            dataFromUrl "https://raw.githubusercontent.com/Monil-H/DataVisCW/main/Data/WFHEngWalesRegions.csv"


        trans =
            transform
                << lookup "properties.label" (londonTravelData []) "RegionName" (luFields [ "Observation","RegionName","MethodCode","TravelMethod" ])

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
