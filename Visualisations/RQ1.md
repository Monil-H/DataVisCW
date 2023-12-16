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

```elm {l}
exampleData =
    dataFromColumns []
        << dataColumn "category" (strs [ "A", "B", "C", "D", "E", "F", "G", "H", "I" ])
        << dataColumn "value" (nums [ 28, 55, 43, 91, 81, 53, 19, 87, 52 ])
```

```elm {v l highlight=[7,8]}
colouredBars : Spec
colouredBars =
    let

        londonTravelData =
            dataFromUrl "https://raw.githubusercontent.com/Monil-H/DataVisCW/main/Data/WFHLondonBorough.csv"

        enc =
            encoding
                << position X [ pName "Borough", pAxis [ axLabelAngle 0 ] ]
                << position Y [ pName "Observation", pNominal, pTitle "categorical value" ]
                << color [ mName "value", mNominal ]
    in
    toVegaLite [ londonTravelData [], enc [], bar [] ]
```

```elm {l v interactive}
londonWFHTotalBar : Spec
londonWFHTotalBar =
    let

        londonTravelData =
            dataFromUrl "https://raw.githubusercontent.com/Monil-H/DataVisCW/main/Data/WFHLondonBorough.csv"

        enc =
            encoding
                << position X [ pName "Borough", pNominal ]
                << position Y [ pName "Observation", pQuant ]
                -- << color
                --     [ mName "value"
                --     , mQuant
                --     , mScale [ scScheme "reds" [] ]
                --     ]
                 << tooltips
                    [[ tName "Borough", tNominal ],
                        [ tName "Observation", tQuant]]
    in
    toVegaLite [ londonTravelData [], enc [], bar [] ]
```

```elm {l v interactive}
engWalesRegionsTotalBar : Spec
engWalesRegionsTotalBar =
    let

        londonTravelData2 =
            dataFromUrl "https://raw.githubusercontent.com/Monil-H/DataVisCW/main/Data/EngWalesRegionsTravelMethods.csv"


        filterTransform =
            transform
                << filter (fiExpr "datum.MethodCode == 12")
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])

        enc =
            encoding
                << position X [ pName "RegionName", pNominal ]
                << position Y [ pName "Observation", pQuant ]
                << color [ mName "TravelMethod", mNominal ]
                 << tooltips
                    [[ tName "RegionName", tNominal ],
                    [tName "MethodCode", tNominal ],
                        [ tName "Observation", tQuant]]
         in
    toVegaLite
        [ width (450 / goldenRatio)
        , height (450 / goldenRatio)
        , cfg []
        , londonTravelData2 []
        , filterTransform []
        , enc []
        , bar [ maFillOpacity 0.75 , maStroke "black",maStrokeWidth 0.2 ]
        ]
```

```elm {l}
goldenRatio : Float
goldenRatio =
    1.618
```

```elm {l v interactive highlight=[7,8-18]}
selRadio : Spec
selRadio =
    let
        ps =
            params
                << param "mySelection"
                    [ paBind
                        (ipRadio
                            [ inName "Crime type:"
                            , inOptions
                                [ "Anti-social behaviour"
                                , "Criminal damage and arson"
                                , "Drugs"
                                , "Robbery"
                                , "Vehicle crime"
                                ]
                            ]
                        )
                    , paSelect sePoint [ seFields [ "crimeType" ] ]
                    ]
    in
    toVegaLite
        [ width 540
        , crimeData -- Data specified in separate code block
        , ps []
        , encHighlight [] -- Encoding specified in separate code block
        , circle []
        ]
```

```elm {l highlight=[21-24,26-29]}
crimeData =
    dataFromUrl "https://gicentre.github.io/data/westMidlands/westMidsCrimesAggregated.tsv" []


crimeColours =
    categoricalDomainMap
        [ ( "Anti-social behaviour", "rgb(59,118,175)" )
        , ( "Burglary", "rgb(81,157,62)" )
        , ( "Criminal damage and arson", "rgb(141,106,184)" )
        , ( "Drugs", "rgb(239,133,55)" )
        , ( "Robbery", "rgb(132,88,78)" )
        , ( "Vehicle crime", "rgb(213,126,190)" )
        ]


encHighlight =
    encoding
        << position X [ pName "month", pTemporal, pAxis [ axTitle "", axGrid False ] ]
        << position Y [ pName "reportedCrimes", pQuant, pAxis [ axGrid False ] ]
        << color
            [ mCondition (prParamEmpty "mySelection")
                [ mName "crimeType", mScale crimeColours ]
                [ mStr "black" ]
            ]
        << opacity
            [ mCondition (prParamEmpty "mySelection")
                [ mNum 1 ]
                [ mNum 0.1 ]
            ]
```

```elm {v interactive}
engWalesRegionsTotalBar2 : Spec
engWalesRegionsTotalBar2 =
    let
        ps =
            params
                << param "mySelection"
                    [ paBind
                        (ipRadio
                            [ inName "MethodCode:"
                            , inOptions
                                [ "1"
                                , "2"
                                , "3"
                                , "4"
                                , "5"
                                , "6"
                                , "7"
                                , "8"
                                , "9"
                                , "10"
                                , "11"
                                , "12"
                                ]
                            ]
                        )
                    , paSelect sePoint [ seFields [ "MethodCode" ] ]
                    ]

        londonTravelData2 =
            dataFromUrl "https://raw.githubusercontent.com/Monil-H/DataVisCW/main/Data/EngWalesRegionsTravelMethods.csv"


        filterTransform =
            transform
                --<< filter (fiExpr "datum.MethodCode == 12")
                --<< filter (fiExpr "datum.MethodCode == 'mySelection'")
        filterTransform2 =
            let
                selectedMethodCodeExpr =
                    "datum.MethodCode == " ++ "mySelection"
            in
            transform
                << filter (fiExpr selectedMethodCodeExpr)

        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])

        encHighlight1 =
            encoding
                << position X [ pName "month", pTemporal, pAxis [ axTitle "", axGrid False ] ]
                << position Y [ pName "reportedCrimes", pQuant, pAxis [ axGrid False ] ]
                -- filter (fiEqual "MethodCode" (fiExpr "mySelection"))
                << color [ mName "MethodCode", mScale crimeColours ]
                << opacity [ mNum 1 ]

        enc =
            encoding
                << position X [ pName "RegionName", pNominal ]
                << position Y [ pName "Observation", pQuant ]
                << color [ mName "TravelMethod", mNominal ]
                << tooltips
                    [[ tName "RegionName", tNominal ],
                    [tName "TravelMethod", tNominal ],
                        [ tName "Observation", tQuant]]
                << color
            [ mCondition (prParamEmpty "mySelection")
                [ mName "MethodCode"]
                [ mStr "black" ]
            ]
                << opacity
            [ mCondition (prParamEmpty "mySelection")
                [ mNum 1 ]
                [ mNum 0.1 ]
            ]
         in
    toVegaLite
        [ width (450 / goldenRatio)
        , height (450 / goldenRatio)
        , cfg []
        , ps []
        , londonTravelData2 []
        , filterTransform []
        , enc []
        , bar [ maFillOpacity 0.75 , maStroke "black",maStrokeWidth 0.2 ]
        ]
```

```elm {l}
data =
    dataFromUrl "https://gicentre.github.io/data/westMidlands/westMidsCrimesAggregated.tsv" []

data2 =
            dataFromUrl "https://raw.githubusercontent.com/Monil-H/DataVisCW/main/Data/EngWalesRegionsTravelMethods.csv"

crimeColours2 =
    categoricalDomainMap
        [ ( "Anti-social behaviour", "rgb(59,118,175)" )
        , ( "Burglary", "rgb(81,157,62)" )
        , ( "Criminal damage and arson", "rgb(141,106,184)" )
        , ( "Drugs", "rgb(239,133,55)" )
        , ( "Robbery", "rgb(132,88,78)" )
        , ( "Vehicle crime", "rgb(213,126,190)" )
        ]

crimeColours3 =
    categoricalDomainMap
        [ ( "Work mainly at or from home", "rgb(59,118,175)" )
        , ( "Underground, metro, light rail, tram", "rgb(81,157,62)" )
        , ( "Train", "rgb(141,106,184)" )
        , ( "Bus, minibus or coach", "rgb(239,133,55)" )
        , ( "Taxi", "rgb(132,88,78)" )
        , ( "Motorcycle, scooter or moped", "rgb(213,126,190)" )
        , ( "Driving a car or van", "rgb(255,0,0)" ) -- You can choose a different color
        , ( "Passenger in a car or van", "rgb(255,255,0)" ) -- You can choose a different color
        , ( "Bicycle", "rgb(0,0,255)" ) -- You can choose a different color
        , ( "On foot", "rgb(255,165,0)" ) -- You can choose a different color
        , ( "Other method of travel to work", "rgb(128,0,128)" ) -- You can choose a different color
        , ( "Not in employment or aged 15 years and under", "rgb(169,169,169)" ) -- You can choose a different color
        ]
```

```elm {v l highlight=9}
facetBars : Spec
facetBars =
    let
        enc =
            encoding
                << position X [ pName "month", pTemporal, pTitle "" ]
                << position Y [ pName "reportedCrimes", pAggregate opSum ]
                << color [ mName "crimeType", mScale crimeColours, mLegend [] ]
                << column [ fName "crimeType", fHeader [ hdTitleFontSize 0 ] ]
    in
    toVegaLite [ height 100, width 120, data, bar [], enc [] ]
```

```elm {v l highlight=9}
facetBars2 : Spec
facetBars2 =
    let
        enc =
            encoding
                << position X [ pName "RegionName", pNominal, pTitle "RegionName" ]
                << position Y [ pName "Observation", pAggregate opSum ]
                << color [ mName "MethodCode", mScale crimeColours3, mLegend [] ]
                << column [ fName "MethodCode", fHeader [ hdTitleFontSize 0 ] ]
    in
    toVegaLite [ height 100, width 120, data2 [], bar [], enc [] ]
```
