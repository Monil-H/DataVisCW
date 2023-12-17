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

```elm {v l interactive}
facetBars2 : Spec
facetBars2 =
    let

        data3 =
            dataFromUrl "https://raw.githubusercontent.com/Monil-H/DataVisCW/main/Data/EngWalesRegionsTravelMethods.csv"

        filterTransform =
            transform
                << filter (fiExpr "datum.MethodCode != 12")
                --<< filter (fiExpr "datum.MethodCode == 'mySelection'")
        enc =
            encoding
             << column [ fName "TravelMethod", fHeader [ hdTitleFontSize 0 ] ]
                << position X [ pName "RegionName", pNominal, pTitle "RegionName" ]
                << position Y [ pName "Observation", pQuant ]
                << color [ mName "TravelMethod", mLegend [] ]
               << tooltips
                    [[ tName "MethodCode", tNominal ],
                        [ tName "Observation", tQuant]]
    in
    toVegaLite [ height 100, width 120, data3 [],filterTransform [], bar [], enc [] ]
```

```elm {v l interactive}
tufteChart : Spec
tufteChart =
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration
                    (coAxis
                        [ axcoTicks False
                        , axcoDomain False
                        , axcoLabelAngle 0
                        , axcoLabelPadding -12
                        ]
                    )

        enc =
            encoding
                << position X
                    [ pName "ageGroup"
                    , pScale [ scPaddingInner 0.5 ]
                    , pAxis [ axTitle "" ]
                    ]
                << position Y
                    [ pName "leave"
                    , pQuant
                    , pAxis [ axTitle "", axValues (nums [ 0.5 ]), axFormat "%" ]
                    ]
        markConfig =
            [ maLabel
                [ leaOffset 2  -- You can adjust the offset value to position the label as desired
                , leaContent ExprData
                ]
            ]

    in
    toVegaLite
        [ width 450
        , height (450 / goldenRatio)
        , cfg []
        , brexitData []
        , enc []
        , bar (maFillOpacity 0.5 :: markConfig)  -- Include the markConfig in the bar function
        ]
```
