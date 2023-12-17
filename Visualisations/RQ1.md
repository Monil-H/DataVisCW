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

### Exploring Travel Methods Within UK and Wales Regions

**Visualisation 1 :** Travel Methods Across Each Region

```elm {l=hidden}

travelMethodsRegionsData =
            dataFromUrl "https://raw.githubusercontent.com/Monil-H/DataVisCW/main/Data/EngWalesRegionsTravelMethods.csv"

regionColour =
    categoricalDomainMap
        [ ( "North East", "rgb(59,118,175)" )
        , ( "North West", "rgb(81,157,62)" )
        , ( "Yorkshire and The Humber", "rgb(141,106,184)" )
        , ( "East Midlands", "rgb(239,133,55)" )
        , ( "West Midlands", "rgb(132,88,78)" )
        , ( "East of England", "rgb(213,126,190)" )
        , ( "London", "rgb(255,0,0)" )
        , ( "South East", "rgb(255,255,0)" )
        , ( "South West", "rgb(0,0,255)" )
        , ( "Wales", "rgb(255,165,0)" )
        ]

travelMethodColour =
    categoricalDomainMap
        [ ( "Work mainly at or from home", "rgb(59,118,175)" )
        , ( "Underground, metro, light rail, tram", "rgb(81,157,62)" )
        , ( "Train", "rgb(141,106,184)" )
        , ( "Bus, minibus or coach", "rgb(239,133,55)" )
        , ( "Taxi", "rgb(132,88,78)" )
        , ( "Motorcycle, scooter or moped", "rgb(213,126,190)" )
        , ( "Driving a car or van", "rgb(255,0,0)" )
        , ( "Passenger in a car or van", "rgb(255,255,0)" )
        , ( "Bicycle", "rgb(0,0,255)" )
        , ( "On foot", "rgb(255,165,0)" )
        , ( "Other method of travel to work", "rgb(128,0,128)" )
        , ( "Not in employment or aged 15 years and under", "rgb(169,169,169)" )
        ]
regionMethodEnc =
    encoding
        << column [ fName "TravelMethod", fHeader [ hdTitleFontSize 0 ] ]
        << position X [ pName "RegionName", pNominal,  pAxis []  ]
        << position Y [ pName "Observation", pQuant , pTitle "" ]
        << color
            [ mCondition (prParam "barSelection")
                [ mName "RegionName", mScale regionColour ]
                [ mStr "rgb(206,194,186)" ]
            ]
        << tooltips
            [[ tName "RegionName", tNominal ],
                [ tName "Observation", tQuant]]
regionMethodparams =
            params
                << param "barSelection" [ paSelect seInterval [ seEncodings [ chX ] ] ]

methodRegionparams =
            params
                << param "barSelection" [ paSelect seInterval [ seEncodings [ chY ] ] ]

methodRegionEnc =
            encoding
             << column [ fName "RegionName", fHeader [ hdTitleFontSize 0 ] ]
                << position X [ pName "TravelMethod", pNominal,  pAxis []]
                << position Y [ pName "Observation", pQuant, pTitle ""  ]
                << color
                    [ mCondition (prParam "barSelection")
                        [ mName "TravelMethod", mScale travelMethodColour ]
                        [ mStr "rgb(206,194,186)" ]
                    ]
               << tooltips
                    [[ tName "TravelMethod", tNominal ],
                        [ tName "Observation", tQuant]]
```

```elm {v interactive}
regionMethodIndividual : Spec
regionMethodIndividual =
    let
        filterTransform =
            transform
                << filter (fiExpr "datum.MethodCode != 7")
                << filter (fiExpr "datum.MethodCode != 12")
                << filter (fiExpr "datum.MethodCode != 1")
                << filter (fiExpr "datum.MethodCode != 5")
                << filter (fiExpr "datum.MethodCode != 6")
                << filter (fiExpr "datum.MethodCode != 11")
    in
    toVegaLite [ height 100, width 120, travelMethodsRegionsData [], filterTransform [], regionMethodparams[] , bar [], regionMethodEnc []]
```

```elm {v interactive}
regionMethodIndividual2 : Spec
regionMethodIndividual2 =
    let
        filterTransform =
            transform
                << filter (fiExpr "datum.MethodCode == 7 || datum.MethodCode == 1" )
    in
    toVegaLite [ height 100, width 120, travelMethodsRegionsData [], filterTransform [], regionMethodparams[] , bar [], regionMethodEnc []]
```

```elm {v interactive}
regionMethodIndividual3 : Spec
regionMethodIndividual3 =
    let
        filterTransform =
            transform
                << filter (fiExpr "datum.MethodCode == 5 || datum.MethodCode == 6 || datum.MethodCode == 11" )
    in
    toVegaLite [ height 100, width 120, travelMethodsRegionsData [], filterTransform [], regionMethodparams[] , bar [], regionMethodEnc []]
```

**Visualisation 2 :** Regional Distribution of Each Travel Method

```elm {v interactive}
methodRegionIndividual : Spec
methodRegionIndividual =
    let
        filterTransform =
            transform
                << filter (fiExpr "datum.MethodCode != 12")
                << filter (fiExpr "datum.RegionCode != 'W92000004'")
                << filter (fiExpr "datum.RegionCode != 'E12000001'")
                << filter (fiExpr "datum.RegionCode != 'E12000003'")
                << filter (fiExpr "datum.RegionCode != 'E12000004'")
                << filter (fiExpr "datum.RegionCode != 'E12000009'")
    in
    toVegaLite [ height 100, width 120, travelMethodsRegionsData [],filterTransform [], regionMethodparams [], bar [], methodRegionEnc [] ]
```

```elm {v interactive}
methodRegionIndividual1 : Spec
methodRegionIndividual1 =
    let
        filterTransform =
            transform
                << filter (fiExpr "datum.MethodCode != 12")
                << filter (fiExpr "datum.RegionCode != 'E12000008'")
                << filter (fiExpr "datum.RegionCode != 'E12000007'")
                << filter (fiExpr "datum.RegionCode != 'E12000002'")
                << filter (fiExpr "datum.RegionCode != 'E12000006'")
                << filter (fiExpr "datum.RegionCode != 'E12000005'")
    in
    toVegaLite [ height 100, width 120, travelMethodsRegionsData [],filterTransform [], regionMethodparams [], bar [], methodRegionEnc [] ]
```
