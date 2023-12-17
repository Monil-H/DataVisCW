---
id: litvis

narrative-schemas:
  - ../narrative-schemas/teaching.yml

elm:
  dependencies:
    gicentre/elm-vegalite: latest
    gicentre/tidy: latest
---

@import "../css/datavis.less"

```elm {l=hidden}
import Tidy exposing (..)
import VegaLite exposing (..)
```

# Working with census tables in your Vega-Lite spec

The data provided by the Office for National Statistics and downloadable from the [census explorer](https://observablehq.com/@jwolondon/census-2021-explorer?collection=@jwolondon/census-2021) may need reshaping before you can use them effectively in your visualizations.

{(infobox|}

This document describes approaches to reshaping that can be achieved from within your elm-VegaLite specifications, but if you find it easier you may prefer to do this either externally (e.g. with a database query or in a spreadsheet), or with Elm and [Tidy](https://package.elm-lang.org/packages/gicentre/tidy/latest/Tidy).

{|infobox)}

## 1. Counts from one or more categories of a census variable

Suppose we wish to look at vehicle ownership from [vehicle table TS045](https://observablehq.com/@jwolondon/census-2021-explorer#TS045) by constituency from the [census explorer](https://observablehq.com/@jwolondon/census-2021-explorer?collection=@jwolondon/census-2021). Selecting `Pivot to wide table` will ensure that the counts for each category in the vehicle table are stored in separate columns.

<img src="https://staff.city.ac.uk/~jwo/datavis2023b/session08/images/censusWide.png" width=450 />

This would allow you to download a CSV table that has this structure:

| code      | areaName            | N/A | v0   | v1    | v2    | v3plus |
| --------- | ------------------- | --- | ---- | ----- | ----- | ------ |
| E14000530 | Aldershot           | 0   | 6802 | 17338 | 14212 | 5252   |
| E14000531 | Aldridge-Brownhills | 0   | 5569 | 12812 | 10447 | 3983   |
| :         | :                   | :   | :    | :     | :     | :      |

`v0` refers to the households with no vehicle access; `v1` access to one car or van; `v2` two cars or vans; and `v3plus` access to three or more cars or vans. The numbers refer to counts of households in a given category and area.

For convenience on this page, this table is stored in `vData` and associated constituency boundaries as `constituencyBoundaries`. Not that because we are going to perform some analysis on the numbers in the dataset, we tell vega-lite explicitly that all category counts (the values in `v0`, `v1` etc.) are numbers with [parse foNum](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#foNum).

```elm {l}
vData : Data
vData =
    dataFromUrl "https://gicentre.github.io/data/census21/tutorial/vehicle_ConstituencyWide.csv"
        [ parse
            -- Ensure all category counts are treated as numbers
            ([ "N/A", "v0", "v1", "v2", "v3plus" ]
                |> List.map (\s -> ( s, foNum ))
            )
        ]


constituencyBoundaries : Data
constituencyBoundaries =
    dataFromUrl "https://gicentre.github.io/data/census21/ewConstituencies.json"
        [ topojsonFeature "constituencies" ]
```

### Option 1.1: Showing a single category

If we want to map the household counts in one of those categories it is a simple case of referencing the column of interest (line 16). In these examples, we'll use that to create choropleth mapping specification with the census table as the primary data source and the geographic boundaries linked as a secondary data source. Here's an example that selects category `v0` (households with no vehicle access) and maps the numbers across parliamentary constituencies:

```elm {l v highlight=16}
mapSingleCat : Spec
mapSingleCat =
    let
        proj =
            projection [ prType transverseMercator, prRotate 2 0 0 ]

        trans =
            -- Census table is the primary data source; link the boundaries to it.
            transform
                << lookup "code" constituencyBoundaries "properties.code" (luAs "geo")

        enc =
            encoding
                << shape [ mName "geo", mGeo ]
                << color
                    [ mName "v0"
                    , mQuant
                    , mLegend [ leOrient loNone, leTitle "Households\nwith no car" ]
                    ]
    in
    toVegaLite [ width 350, height 400, vData, trans [], proj, enc [], geoshape [] ]
```

### Option 1.2: Adding multiple categories

If it makes sense to add them together, perhaps you wish to combine two or more categories. For example, we might want to map the numbers of households with access to two or more vehicles, which would involve adding the counts from categories `v2` and `v3plus`. Here it is much the same as the previous example accept we use the [calculateAs transform](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#3-5-data-calculation) to create a new field based on the sum of the counts in `v2` and `v3plus` (line 11).

```elm {l v highlight=[11,17]}
mapTwoCats : Spec
mapTwoCats =
    let
        proj =
            projection [ prType transverseMercator, prRotate 2 0 0 ]

        trans =
            transform
                << lookup "code" constituencyBoundaries "properties.code" (luAs "geo")
                -- Create new field from two existing ones
                << calculateAs "datum.v2 + datum.v3plus" "cars2plus"

        enc =
            encoding
                << shape [ mName "geo", mGeo ]
                << color
                    [ mName "cars2plus"
                    , mQuant
                    , mLegend
                        [ leOrient loNone
                        , leTitle "Households with\naccess to two\nor more vehicles"
                        ]
                    ]
    in
    toVegaLite [ width 350, height 400, vData, trans [], proj, enc [], geoshape [] ]
```

## 2. Converting counts into proportions

There's a [danger in plotting absolute numbers in a choropleth map](https://www.esri.com/arcgis-blog/products/product/mapping/mapping-coronavirus-responsibly/) (like counts of households). We don't know if high values are because there is a high proportion of the category we are mapping, or simply because lots of people happen to live there.

More reliable is to calculate the _proportion_ of households (or people). The census table doesn't have that proportion directly, but we can calculate it by comparing the category of interest with the sum of all the category counts (i.e. total household count). For example, to calculate the proportion of households with no access to a vehicle:

$pnocar = \frac {v0} {v0 + v1 + v2 + v3plus}$

Again we can use the [calculateAs transform](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#3-5-data-calculation) to perform this calculation:

```elm {l v highlight=[11,17]}
mapPercCats : Spec
mapPercCats =
    let
        proj =
            projection [ prType transverseMercator, prRotate 2 0 0 ]

        trans =
            transform
                << lookup "code" constituencyBoundaries "properties.code" (luAs "geo")
                -- Calculate proportion
                << calculateAs "datum.v0 / (datum.v0 + datum.v1 + datum.v2 + datum.v3plus)" "pNoCar"

        enc =
            encoding
                << shape [ mName "geo", mGeo ]
                << color
                    [ mName "pNoCar"
                    , mQuant
                    , mLegend
                        [ leOrient loNone
                        , leTitle "% households with\nno access a vehicle"
                        , leFormat "%"
                        ]
                    ]
    in
    toVegaLite [ width 350, height 400, vData, trans [], proj, enc [], geoshape [] ]
```

## 3. Statistical Summaries

If you want to aggregate the categories in some other way, for example, to show the most common category in each area, we can benefit from an alternative approach.

First we need to ensure the data are _tidy_. If you download the census data _without_ the `Pivot to wide table` selected, a single tidy table is generated.

<img src="https://staff.city.ac.uk/~jwo/datavis2023b/session08/images/censusWide.png" width=450 />

For the vehicle data, the downloaded table would have this form which you should recognise as _tidy_:

| code      | category | value | areaName            |
| --------- | -------- | ----- | ------------------- |
| E14000530 | N/A      | 0     | Aldershot           |
| E14000530 | v0       | 6802  | Aldershot           |
| E14000530 | v1       | 17338 | Aldershot           |
| E14000530 | v2       | 14212 | Aldershot           |
| E14000530 | v3plus   | 5252  | Aldershot           |
| E14000531 | N/A      | 0     | Aldridge-Brownhills |
| E14000531 | v0       | 5569  | Aldridge-Brownhills |
| E14000531 | v1       | 12812 | Aldridge-Brownhills |
| E14000531 | v2       | 10447 | Aldridge-Brownhills |
| E14000531 | v3plus   | 3983  | Aldridge-Brownhills |
| :         | :        | :     | :                   |

```elm {l}
vTidyData : Data
vTidyData =
    dataFromUrl "https://gicentre.github.io/data/census21/tutorial/vehicle_Constituency.csv"
        -- Ensure counts are treated as numbers
        [ parse [ ( "value", foNum ) ] ]
```

### 3.1 Spread of values in each area

Now we have the count data in a single column we can perform aggregation operations on it to turn the multiple values for each area into some single summary value. For example, to show the spread in values across categories (do all households have similar vehicle access or is there big variation across households?) we could use the [opStdev aggregation operator](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#opStdev) as part of a data transform:

```elm {l v highlight=[9,16]}
mapStdevCat : Spec
mapStdevCat =
    let
        proj =
            projection [ prType transverseMercator, prRotate 2 0 0 ]

        trans =
            transform
                << aggregate [ opAs opStdev "value" "spread" ] [ "code" ]
                << lookup "code" constituencyBoundaries "properties.code" (luAs "geo")

        enc =
            encoding
                << shape [ mName "geo", mGeo ]
                << color
                    [ mName "spread"
                    , mQuant
                    , mLegend
                        [ leOrient loNone
                        , leTitle "Variation (stdev)\nin vehicle access"
                        ]
                    ]
    in
    toVegaLite [ width 350, height 400, vTidyData, trans [], proj, enc [], geoshape [] ]
```

Showing the standard deviation as above gives some sense of how much variation there is in counts for each vehicle access category, but it does suffer from the problem of unequal numbers of households in each constituency. Is a larger standard deviation because there is genuinely more variation or a function of the population of an area?

We can standardise the measure of spread by dividing it by the average number of households in each area. This is achieved by calculating a mean aggregation and dividing the spread by that mean, giving the [coefficient of variation](https://en.wikipedia.org/wiki/Coefficient_of_variation):

```elm {l v highlight=[9,10,17]}
mapCovCat : Spec
mapCovCat =
    let
        proj =
            projection [ prType transverseMercator, prRotate 2 0 0 ]

        trans =
            transform
                << aggregate [ opAs opStdev "value" "spread", opAs opMean "value" "mean" ] [ "code" ]
                << calculateAs "datum.spread / datum.mean" "cov"
                << lookup "code" constituencyBoundaries "properties.code" (luAs "geo")

        enc =
            encoding
                << shape [ mName "geo", mGeo ]
                << color
                    [ mName "cov"
                    , mQuant
                    , mLegend
                        [ leOrient loNone
                        , leTitle "Variation (cov)\nin vehicle access"
                        ]
                    ]
    in
    toVegaLite [ width 350, height 400, vTidyData, trans [], proj, enc [], geoshape [] ]
```

### 3.2 Most common category in each area

For any given area, which category has the most number of people within it? This is another way of summarising the distribution of counts across all categories. Again we can use an aggregation operator to find the category with the maximum count associated with it. You may be tempted to use [opMax](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#opMax), but if we applied this to our `value` column, it would generate the maximum count in each area. What we want is the category associated with that maximum count. For this we use [opArgMax](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#opArgMax). This will generate a list of all fields associated with the most common value, from which we can access the category name (line 16):

```elm {l v highlight=[9,16]}
mapModalCat : Spec
mapModalCat =
    let
        proj =
            projection [ prType transverseMercator, prRotate 2 0 0 ]

        trans =
            transform
                << aggregate [ opAs (opArgMax Nothing) "value" "mostCommon" ] [ "code" ]
                << lookup "code" constituencyBoundaries "properties.code" (luAs "geo")

        enc =
            encoding
                << shape [ mName "geo", mGeo ]
                << color
                    [ mName "mostCommon['category']"
                    , mOrdinal
                    , mLegend
                        [ leOrient loNone
                        , leTitle "Most common form\nof vehicle access"
                        ]
                    ]
    in
    toVegaLite [ width 350, height 400, vTidyData, trans [], proj, enc [], geoshape [] ]
```

### 3.3 Averages across categories

What is the average number of vehicles accessible to households within an area? This may be a reasonable question to ask, but quite hard to calculate accurately from a set of categorical counts such as the census.

If we were simply to calculate the mean (for example using an [opMean aggregation](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#opMean)), this would generate a number for each area, but it would just be the average of all the counts in each category (i.e. the total number of households in the area).

What we want is the average of 0, 1, 2 and 3 (vehicles) weighted according to the counts in each of those categories. Even then, it would only be an approximation as the `v3plus` category might contain households with 3, 4 or 27 cars. Nevertheless, it may provide an approximation that is useful to us.

To do this, we need to return to the 'wide' (non-tidy) version of the table and perform another custom calculation in the form:

$wa = \frac{0 \times v0 + 1 \times v1 + 2 \times v2 + 3 \times v3plus}{v0 + v1 + v2 + v3plus}$

which simplifies to

$wa = \frac{v1 + 2 \times v2 + 3 \times v3plus}{v0 + v1 + v2 + v3plus}$

_Note: this only makes sense for categories that are themselves representing numeric quantities (number of vehicles in this case), and only then if they are approximately linear (0,1,2 and to some extent 3)._

```elm {l v highlight=[11,17]}
avAcrossCats : Spec
avAcrossCats =
    let
        proj =
            projection [ prType transverseMercator, prRotate 2 0 0 ]

        trans =
            transform
                << lookup "code" constituencyBoundaries "properties.code" (luAs "geo")
                -- Calculate weighted average
                << calculateAs "(datum.v1+2*datum.v2+3*datum.v3plus) / (datum.v0 + datum.v1 + datum.v2 + datum.v3plus)" "wa"

        enc =
            encoding
                << shape [ mName "geo", mGeo ]
                << color
                    [ mName "wa"
                    , mQuant
                    , mLegend
                        [ leOrient loNone
                        , leTitle "Approximate average\nnumber of vehicles\nper household"
                        ]
                    ]
    in
    toVegaLite [ width 350, height 400, vData, trans [], proj, enc [], geoshape [] ]
```
