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

```elm {interactive v highlight=15}
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


        trans2 =
            transform
                -- << aggregate [ opAs (opArgMax Nothing) "observation" "mostCommon" ] [ "code" ]
                << lookup "id" (londonTravelData2 []) "Borough" (luFields [ "Observation","Borough","MethodCode","TravelMethod", "RegionCode" ])
                --<< filter (fiExpr "datum.Borough == 'Barnet'")

        proj =
            projection [ prType transverseMercator, prRotate 2 0 0 ]

        enc =
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
        , trans2 []
        , proj
        , enc []
        , geoshape [ maStroke "white" ]
        ]
```

```elm {interactive  v }
londonChoropleth2 : Spec
londonChoropleth2 =
    let
        geoData =
            dataFromUrl "https://gicentre.github.io/data/geoTutorials/londonBoroughs.json"
                [ topojsonFeature "boroughs" ]

        londonTravelData =
            dataFromUrl "https://raw.githubusercontent.com/Monil-H/DataVisCW/main/Data/LondonBoroughTravelMethods.csv"

        londonTravelData2 =
            dataFromUrl "https://raw.githubusercontent.com/Monil-H/DataVisCW/main/Data/MaxTravelMethodLondon.csv"


        trans2 =
            transform
                -- << aggregate [ opAs (opArgMax Nothing) "observation" "mostCommon" ] [ "code" ]
                << lookup "id" (londonTravelData2 []) "Borough" (luFields [ "Observation","Borough","MethodCode","TravelMethod", "RegionCode" ])
                --<< filter (fiExpr "datum.Borough == 'Barnet'")

        proj =
            projection [ prType transverseMercator, prRotate 2 0 0 ]

        enc =
            encoding
                << color
                    [ mName "TravelMethod"
                    , mNominal
                    , mTitle "Most Used Transport Method"
                    , mScale [ scScheme "dark2" [] ]
                    ]
                << tooltips
                    [ [  tName "Borough",tNominal ]
                    , [ tName "TravelMethod" ,tNominal]
                    , [ tName "Observation", tQuant]
                    ]

        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coLegend [ lecoOrient loBottomRight, lecoOffset -25 ])
    in
    toVegaLite
        [ width 600
        , height 450
        , cfg []
        , geoData
        , trans2 []
        , proj
        , enc []
        , geoshape [ maStroke "white" ]
        ]
```

```elm {interactive l v highlight=15}
londonBoroughsChoropleth : Spec
londonBoroughsChoropleth =
    let
        londonGeoData =
            dataFromUrl "https://gicentre.github.io/data/geoTutorials/londonBoroughs.json"
                [ topojsonFeature "boroughs" ]

        londonMostTravelData =
            dataFromUrl "https://raw.githubusercontent.com/Monil-H/DataVisCW/main/Data/MaxTravelMethodLondon.csv"


        trans2 =
            transform
                -- << aggregate [ opAs (opArgMax Nothing) "observation" "mostCommon" ] [ "code" ]
                << lookup "id" (londonMostTravelData []) "Borough" (luFields [ "Observation","Borough","MethodCode","TravelMethod", "RegionCode" ])

        proj =
            projection [ prType transverseMercator, prRotate 2 0 0 ]

        enc =
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

        ps =
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

    in
    toVegaLite
        [ width 600
        , height 450
        , cfg []
        , londonGeoData
        , trans2 []
        , proj
        , ps []
        , enc []
        , geoshape [ maStroke "white" ]
        ]
```
