---
id: litvis

narrative-schemas:
  - ../../lectures/narrative-schemas/project.yml

elm:
  dependencies:
    gicentre/elm-vegalite: latest
    gicentre/tidy: latest
---

```elm {l=hidden}
import Tidy exposing (..)
import VegaLite exposing (..)
```

# Data Visualization Project Summary

Research Topic: Identifying London's Employment Trends: The Effects of Work History, Gender, Age, and Borough.

- Question 1: Based on demographic characteristics including age, gender, race, and religion, what are the main job trends in London?

- Question 2: How do employment rates vary across different London boroughs, and what are the differences between genders?

- Question 3: How do employment rates vary by age and sex across London’s general population?

## Visualization 1: England And Wales Choropleth Map

```elm {l=hidden}
vData : Data
vData =
    dataFromUrl "https://raw.githubusercontent.com/VithujaThangarajah/data/refs/heads/main/employmentHistory_Constituency.csv"
        [ parse
            -- Ensure all category counts are treated as numbers
            ([ "N/A", "worked12", "notWorked12", "neverWorked" ]
                |> List.map (\s -> ( s, foNum ))
            )
        ]


constituencyBoundaries : Data
constituencyBoundaries =
    dataFromUrl "https://gicentre.github.io/data/census21/ewConstituencies.json"
        [ topojsonFeature "constituencies" ]
```

```elm {v interactive}
workedmapChoropleth : Spec
workedmapChoropleth =
    let
        geoData =
            dataFromUrl "https://gicentre.github.io/data/census21/ewConstituencies.json"
                [ topojsonFeature "constituencies" ]

        proj =
            projection [ prType transverseMercator, prRotate 2 0 0 ]

        trans =
            transform
                << lookup "code" constituencyBoundaries "properties.code" (luAs "geo")

        enc =
            encoding
                << shape [ mName "geo", mGeo ]
                << color
                    [ mName "worked12"
                    , mQuant
                    , mScale [ scScheme "blues" [] ]
                    , mLegend [ leOrient loRight, leTitle "% Worked in 12 Months", leFormat "%" ]
                    ]
    in
    toVegaLite [ width 250, height 300, vData, trans [], proj, enc [], geoshape [] ]


neverworkedmapChoropleth : Spec
neverworkedmapChoropleth =
    let
        geoData =
            dataFromUrl "https://gicentre.github.io/data/census21/ewConstituencies.json"
                [ topojsonFeature "constituencies" ]

        proj =
            projection [ prType transverseMercator, prRotate 2 0 0 ]

        trans =
            transform
                << lookup "code" constituencyBoundaries "properties.code" (luAs "geo")

        enc =
            encoding
                << shape [ mName "geo", mGeo ]
                << color
                    [ mName "neverWorked"
                    , mQuant
                    , mScale [ scScheme "blues" [] ]
                    , mLegend [ leOrient loRight, leTitle "% Worked in 12 Months", leFormat "%" ]
                    ]
    in
    toVegaLite [ width 250, height 300, vData, trans [], proj, enc [], geoshape [] ]


notworkedmapChoropleth : Spec
notworkedmapChoropleth =
    let
        geoData =
            dataFromUrl "https://gicentre.github.io/data/census21/ewConstituencies.json"
                [ topojsonFeature "constituencies" ]

        proj =
            projection [ prType transverseMercator, prRotate 2 0 0 ]

        trans =
            transform
                << lookup "code" constituencyBoundaries "properties.code" (luAs "geo")

        enc =
            encoding
                << shape [ mName "geo", mGeo ]
                << color
                    [ mName "notWorked12"
                    , mQuant
                    , mScale [ scScheme "blues" [] ]
                    , mLegend [ leOrient loRight, leTitle "% Worked in 12 Months", leFormat "%" ]
                    ]
    in
    toVegaLite [ width 250, height 300, vData, trans [], proj, enc [], geoshape [] ]
```

## Regional Choropleth Map

Unemployment rates for 16-year-olds and older varied by local authority in 2021:

```elm {v interactive}
employedChoropleth : Spec
employedChoropleth =
    let
        geoData =
            dataFromUrl "https://gicentre.github.io/data/geoTutorials/londonBoroughs.json"
                [ topojsonFeature "boroughs" ]

        employedData =
            dataFromColumns []
                << dataColumn "borough" (strColumn "borough" employedTable |> strs)
                << dataColumn "percent" (numColumn "percent" employedTable |> nums)

        trans =
            transform
                << lookup "id" (employedData []) "borough" (luFields [ "percent" ])

        proj =
            projection [ prType transverseMercator, prRotate 2 0 0 ]

        enc =
            encoding
                << color
                    [ mName "percent"
                    , mQuant
                    , mScale [ scScheme "blues" [] ]
                    , mTitle "Employment (%)"
                    ]
                << tooltips [ [ tName "id", tTitle "Borough" ], [ tName "percent", tFormat ".1f", tTitle "Employment (%)" ] ]

        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coLegend [ lecoOrient loBottomRight, lecoOffset 0 ])
    in
    toVegaLite
        [ width 400
        , height 350
        , cfg []
        , geoData
        , trans []
        , proj
        , enc []
        , geoshape []
        ]


unemployedChoropleth : Spec
unemployedChoropleth =
    let
        geoData =
            dataFromUrl "https://gicentre.github.io/data/geoTutorials/londonBoroughs.json"
                [ topojsonFeature "boroughs" ]

        unemployedData =
            dataFromColumns []
                << dataColumn "borough" (strColumn "borough" unemployedTable |> strs)
                << dataColumn "percent" (numColumn "percent" unemployedTable |> nums)

        trans =
            transform
                << lookup "id" (unemployedData []) "borough" (luFields [ "percent" ])

        proj =
            projection [ prType transverseMercator, prRotate 2 0 0 ]

        enc =
            encoding
                << color
                    [ mName "percent"
                    , mQuant
                    , mScale [ scScheme "reds" [] ]
                    , mTitle "Unemployment (%)"
                    ]
                << tooltips [ [ tName "id", tTitle "Borough" ], [ tName "percent", tFormat ".1f", tTitle "Unemployment (%)" ] ]

        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coLegend [ lecoOrient loBottomRight, lecoOffset 0 ])
    in
    toVegaLite
        [ width 400
        , height 350
        , cfg []
        , geoData
        , trans []
        , proj
        , enc []
        , geoshape []
        ]
```

## Visualization 2: Stacked Bar Chart

A greater proportion of men were employed than women in all local authorities in England and Wales

```elm {l=hidden}
menemploymentproportionData =
    dataFromColumns []
        << dataColumn "London Boroughs" (strs [ "Hounslow", "Kensington and Chelsea", "City of London", "Westminster", "Wandsworth", "Redbridge", "Havering", "Harrow", "Hammersmith and Fulham", "Brent", "Barnet", "Ealing", "Waltham Forest", "Tower Hamlets", "Sutton", "Southwark", "Richmond upon Thames", "Newham", "Merton", "Lewisham", "Lambeth", "Kingston upon Thames", "Islington", "Hillingdon", "Hackney", "Haringey", "Greenwich", "Enfield", "Croydon", "Camden", "Bromley", "Bexley", "Barking and Dagenham" ])
        << dataColumn "Subgroup" (strs [ "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men" ])
        << dataColumn "Percentage" (nums [ 66.7, 61.3, 70.3, 62.4, 73.2, 65.5, 64.6, 66.0, 66.5, 64.0, 65.4, 64.8, 67.7, 68.0, 67.9, 68.8, 67.4, 66.4, 69.9, 66.9, 70.5, 67.0, 66.1, 64.5, 66.4, 65.7, 66.2, 61.1, 64.8, 62.2, 65.6, 64.7, 64.8 ])
```

```elm {v interactive}
menemploymentproportionChart : Spec
menemploymentproportionChart =
    let
        trans =
            transform
                << lookup "id" (menemploymentproportionData []) "Percentage" (luFields [ "percent" ])

        enc =
            encoding
                << position X
                    [ pName "London Boroughs"
                    , pOrdinal
                    , pAxis [ axTitle "Proportion Of Men In Employment", axLabelAngle -45 ]
                    ]
                << position Y
                    [ pName "Percentage"
                    , pQuant
                    , pAxis [ axTitle "Percentage" ]
                    ]
                << color
                    [ mName "London Boroughs"
                    , mNominal
                    , mLegend [ leTitle "Men In Employment" ]
                    , mScale
                        [ scScheme "blues" [], scDomain (doMid 2) ]
                    ]
                << tooltips [ [ tName "Percentage", tTitle "Percentage (%)" ] ]
    in
    toVegaLite
        [ menemploymentproportionData []
        , enc []
        , bar []
        , height 300
        , width 400
        ]
```

```elm {l=hidden}
womenemploymentproportionData =
    dataFromColumns []
        << dataColumn "London Boroughs" (strs [ "Hounslow", "Kensington and Chelsea", "City of London", "Westminster", "Wandsworth", "Redbridge", "Havering", "Harrow", "Hammersmith and Fulham", "Brent", "Barnet", "Ealing", "Waltham Forest", "Tower Hamlets", "Sutton", "Southwark", "Richmond upon Thames", "Newham", "Merton", "Lewisham", "Lambeth", "Kingston upon Thames", "Islington", "Hillingdon", "Hackney", "Haringey", "Greenwich", "Enfield", "Croydon", "Camden", "Bromley", "Bexley", "Barking and Dagenham" ])
        << dataColumn "Subgroup" (strs [ "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women" ])
        << dataColumn "Percentage" (nums [ 54.9, 52.2, 62.6, 52.6, 66.5, 51.8, 54.9, 53.6, 60.8, 53.3, 55.4, 53.9, 58.9, 54.9, 58.4, 61.9, 58.3, 52.0, 60.2, 61.9, 65.6, 57.6, 59.5, 53.8, 62.1, 59.0, 58.5, 51.4, 57.6, 53.6, 57.2, 56.2, 52.7 ])
```

```elm {v interactive}
womenemploymentproportionChart : Spec
womenemploymentproportionChart =
    let
        trans =
            transform
                << lookup "id" (womenemploymentproportionData []) "Percentage" (luFields [ "percent" ])

        enc =
            encoding
                << position X
                    [ pName "London Boroughs"
                    , pOrdinal
                    , pAxis [ axTitle "Proportion Of Women In Employment", axLabelAngle -45 ]
                    ]
                << position Y
                    [ pName "Percentage"
                    , pQuant
                    , pAxis [ axTitle "Percentage" ]
                    ]
                << color
                    [ mName "London Boroughs"
                    , mNominal
                    , mLegend [ leTitle "Women In Employment" ]
                    , mScale
                        [ scScheme "reds" [], scDomain (doMid 2) ]
                    ]
                << tooltips [ [ tName "Percentage", tTitle "Percentage (%)" ] ]
    in
    toVegaLite
        [ womenemploymentproportionData []
        , enc []
        , bar []
        , height 300
        , width 400
        ]
```

```elm {l=hidden}
diffemploymentproportionData =
    dataFromColumns []
        << dataColumn "London Boroughs" (strs [ "Hounslow", "Kensington and Chelsea", "City of London", "Westminster", "Wandsworth", "Redbridge", "Havering", "Harrow", "Hammersmith and Fulham", "Brent", "Barnet", "Ealing", "Waltham Forest", "Tower Hamlets", "Sutton", "Southwark", "Richmond upon Thames", "Newham", "Merton", "Lewisham", "Lambeth", "Kingston upon Thames", "Islington", "Hillingdon", "Hackney", "Haringey", "Greenwich", "Enfield", "Croydon", "Camden", "Bromley", "Bexley", "Barking and Dagenham" ])
        << dataColumn "Subgroup" (strs [ "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference" ])
        << dataColumn "Percentage" (nums [ 11.8, 9.1, 7.7, 9.8, 6.7, 13.7, 9.7, 12.4, 5.7, 10.7, 10.0, 10.9, 8.8, 13.1, 9.5, 6.9, 9.1, 14.4, 9.7, 5, 4.9, 9.4, 6.6, 10.7, 4.3, 6.7, 7.7, 9.7, 7.2, 8.6, 8.4, 8.5, 12.1 ])
```

```elm {v interactive}
diffemploymentproportionChart : Spec
diffemploymentproportionChart =
    let
        trans =
            transform
                << lookup "id" (diffemploymentproportionData []) "Percentage" (luFields [ "percent" ])

        enc =
            encoding
                << position X
                    [ pName "London Boroughs"
                    , pOrdinal
                    , pAxis [ axTitle "Difference Of Men And Women In Employment", axLabelAngle -45 ]
                    ]
                << position Y
                    [ pName "Percentage"
                    , pQuant
                    , pAxis [ axTitle "Percentage" ]
                    ]
                << color
                    [ mName "London Boroughs"
                    , mNominal
                    , mLegend [ leTitle "Men And Women In Employment" ]
                    , mScale
                        [ scScheme "redblue" [], scDomain (doMid 2) ]
                    ]
                << tooltips [ [ tName "Percentage", tTitle "Percentage (%)" ] ]
    in
    toVegaLite
        [ diffemploymentproportionData []
        , enc []
        , bar []
        , height 300
        , width 400
        ]
```

```elm {l=hidden}
menunemployedproportionData =
    dataFromColumns []
        << dataColumn "London Boroughs" (strs [ "Hounslow", "Kensington and Chelsea", "City of London", "Westminster", "Wandsworth", "Redbridge", "Havering", "Harrow", "Hammersmith and Fulham", "Brent", "Barnet", "Ealing", "Waltham Forest", "Tower Hamlets", "Sutton", "Southwark", "Richmond upon Thames", "Newham", "Merton", "Lewisham", "Lambeth", "Kingston upon Thames", "Islington", "Hillingdon", "Hackney", "Haringey", "Greenwich", "Enfield", "Croydon", "Camden", "Bromley", "Bexley", "Barking and Dagenham" ])
        << dataColumn "Subgroup" (strs [ "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men", "Men" ])
        << dataColumn "Percentage" (nums [ 5.5, 4.7, 4.1, 5, 4.3, 4.9, 4, 4.2, 5.3, 5.9, 4.7, 5.6, 5.7, 6.5, 3.7, 5.9, 3.7, 6.7, 4.4, 6.4, 5.9, 4.2, 5.9, 5.2, 6.1, 6.2, 5.8, 5.3, 5.3, 5.5, 3.7, 4, 6.2 ])
```

```elm {v interactive}
menunemployedproportionChart : Spec
menunemployedproportionChart =
    let
        trans =
            transform
                << lookup "id" (menunemployedproportionData []) "Percentage" (luFields [ "percent" ])

        enc =
            encoding
                << position X
                    [ pName "London Boroughs"
                    , pOrdinal
                    , pAxis [ axTitle "Proportion Of Men Unemployed", axLabelAngle -45 ]
                    ]
                << position Y
                    [ pName "Percentage"
                    , pQuant
                    , pAxis [ axTitle "Percentage" ]
                    ]
                << color
                    [ mName "London Boroughs"
                    , mNominal
                    , mLegend [ leTitle "Men Unemployed" ]
                    , mScale
                        [ scScheme "blues" [], scDomain (doMid 2) ]
                    ]
                << tooltips [ [ tName "Percentage", tTitle "Percentage (%)" ] ]
    in
    toVegaLite
        [ menunemployedproportionData []
        , enc []
        , bar []
        , height 300
        , width 400
        ]
```

```elm {l=hidden}
womenunemployedproportionData =
    dataFromColumns []
        << dataColumn "London Boroughs" (strs [ "Hounslow", "Kensington and Chelsea", "City of London", "Westminster", "Wandsworth", "Redbridge", "Havering", "Harrow", "Hammersmith and Fulham", "Brent", "Barnet", "Ealing", "Waltham Forest", "Tower Hamlets", "Sutton", "Southwark", "Richmond upon Thames", "Newham", "Merton", "Lewisham", "Lambeth", "Kingston upon Thames", "Islington", "Hillingdon", "Hackney", "Haringey", "Greenwich", "Enfield", "Croydon", "Camden", "Bromley", "Bexley", "Barking and Dagenham" ])
        << dataColumn "Subgroup" (strs [ "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women", "Women" ])
        << dataColumn "Percentage" (nums [ 4.9, 4.5, 4.1, 4.6, 3.7, 4.1, 3.4, 3.8, 4.6, 5.1, 4.1, 4.7, 4.6, 5.5, 3.1, 5.3, 3.1, 5.8, 3.6, 5.1, 4.8, 3.5, 5, 4.4, 4.9, 5, 4.9, 4.2, 4.2, 4.6, 3.1, 3.5, 5.1 ])
```

```elm {v interactive}
womenunemployedproportionChart : Spec
womenunemployedproportionChart =
    let
        trans =
            transform
                << lookup "id" (womenunemployedproportionData []) "Percentage" (luFields [ "percent" ])

        enc =
            encoding
                << position X
                    [ pName "London Boroughs"
                    , pOrdinal
                    , pAxis [ axTitle "Proportion Of Women Unemployed", axLabelAngle -45 ]
                    ]
                << position Y
                    [ pName "Percentage"
                    , pQuant
                    , pAxis [ axTitle "Percentage" ]
                    ]
                << color
                    [ mName "London Boroughs"
                    , mNominal
                    , mLegend [ leTitle "Women Unemployed" ]
                    , mScale
                        [ scScheme "purples" [], scDomain (doMid 2) ]
                    ]
                << tooltips [ [ tName "Percentage", tTitle "Percentage (%)" ] ]
    in
    toVegaLite
        [ womenunemployedproportionData []
        , enc []
        , bar []
        , height 300
        , width 400
        ]
```

```elm {l=hidden}
diffunemployedproportionData =
    dataFromColumns []
        << dataColumn "London Boroughs" (strs [ "Hounslow", "Kensington and Chelsea", "City of London", "Westminster", "Wandsworth", "Redbridge", "Havering", "Harrow", "Hammersmith and Fulham", "Brent", "Barnet", "Ealing", "Waltham Forest", "Tower Hamlets", "Sutton", "Southwark", "Richmond upon Thames", "Newham", "Merton", "Lewisham", "Lambeth", "Kingston upon Thames", "Islington", "Hillingdon", "Hackney", "Haringey", "Greenwich", "Enfield", "Croydon", "Camden", "Bromley", "Bexley", "Barking and Dagenham" ])
        << dataColumn "Subgroup" (strs [ "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference", "Difference" ])
        << dataColumn "Percentage" (nums [ 0.6, 0.2, 0, 0.4, 0.6, 0.8, 0.6, 0.5, 0.7, 0.8, 0.6, 0.9, 1.1, 1, 0.6, 0.6, 0.6, 0.9, 0.8, 1.3, 1.1, 0.7, 0.9, 0.8, 1.2, 1.2, 0.9, 1.1, 1.1, 0.9, 0.6, 0.5, 1.1 ])
```

```elm {v interactive}
diffunemployedproportionChart : Spec
diffunemployedproportionChart =
    let
        trans =
            transform
                << lookup "id" (diffunemployedproportionData []) "Percentage" (luFields [ "percent" ])

        enc =
            encoding
                << position X
                    [ pName "London Boroughs"
                    , pOrdinal
                    , pAxis [ axTitle "Difference Of Men And Women Unemployed", axLabelAngle -45 ]
                    ]
                << position Y
                    [ pName "Percentage"
                    , pQuant
                    , pAxis [ axTitle "Percentage" ]
                    ]
                << color
                    [ mName "London Boroughs"
                    , mNominal
                    , mLegend [ leTitle "Men And Women Unemployed" ]
                    , mScale
                        [ scScheme "bluepurple" [], scDomain (doMid 2) ]
                    ]
                << tooltips [ [ tName "Percentage", tTitle "Percentage (%)" ] ]
    in
    toVegaLite
        [ diffunemployedproportionData []
        , enc []
        , bar []
        , height 300
        , width 400
        ]
```

## Visualization 3: Stacked Bar Chart

Census 2021, employment rates for 16-year-olds and older by age and sex:

```elm {l=hidden}
genderData =
    dataFromColumns []
        << dataColumn "Sex" (strs [ "Female", "Male", "All Adults", "Female", "Male", "All Adults", "Female", "Male", "All Adults", "Female", "Male", "All Adults", "Female", "Male", "All Adults", "Female", "Male", "All Adults", "Female", "Male", "All Adults", "Female", "Male", "All Adults", "Female", "Male", "All Adults", "Female", "Male", "All Adults", "Female", "Male", "All Adults", "Female", "Male", "All Adults", "Female", "Male", "All Adults" ])
        << dataColumn "Age Group" (strs [ "16 to 19", "16 to 19", "16 to 19", "20 to 24", "20 to 24", "20 to 24", "25 to 29", "25 to 29", "25 to 29", "30 to 34", "30 to 34", "30 to 34", "35 to 39", "35 to 39", "35 to 39", "40 to 44", "40 to 44", "40 to 44", "45 to 49", "45 to 49", "45 to 49", "50 to 54", "50 to 54", "50 to 54", "55 to 59", "55 to 59", "55 to 59", "60 to 64", "60 to 64", "60 to 64", "65 to 69", "65 to 69", "65 to 69", "70 to 74", "70 to 74", "70 to 74", "75 and over", "75 and over", "75 and over" ])
        << dataColumn "Employment Percentage" (nums [ 24.5, 23.7, 24.1, 60.6, 61.3, 61.0, 76.6, 81.6, 79.0, 75.5, 85.2, 80.2, 75.0, 86.0, 80.3, 76.1, 85.7, 80.8, 76.7, 84.7, 80.6, 74.9, 82.5, 78.7, 67.2, 76.2, 71.6, 49.7, 61.3, 55.4, 20.0, 29.8, 24.7, 7.0, 12.2, 9.5, 2.2, 4.8, 3.3 ])
```

```elm {v interactive}
genderChart : Spec
genderChart =
    let
        enc =
            encoding
                << position X
                    [ pName "Age Group"
                    , pOrdinal
                    , pAxis [ axTitle "Age Group", axLabelAngle -45 ]
                    ]
                << position Y
                    [ pName "Employment Percentage"
                    , pQuant
                    , pAxis [ axTitle "Employment Percentage" ]
                    ]
                << color
                    [ mName "Sex"
                    , mNominal
                    , mLegend [ leTitle "Sex" ]
                    , mScale [ scRange (raStrs [ "#b3cde3", "#8c96c6", "#8856a7" ]) ] -- Light blue to purple
                    ]
                << tooltips [ [ tName "Employment Percentage", tTitle "Percentage(%)" ] ]
    in
    toVegaLite
        [ genderData []
        , enc []
        , bar []
        , height 400
        , width 400
        ]
```

## Visualization 4: By religion and gender, the employment percentages

```elm {l=hidden v interactive highlight=[7,8-18]}
religionselRadio : Spec
religionselRadio =
    let
        ps =
            params
                << param "mySelection"
                    [ paBind
                        (ipRadio
                            [ inName "Sex:"
                            , inOptions
                                [ "Female"
                                , "Male"
                                ]
                            ]
                        )
                    , paSelect sePoint [ seFields [ "Sex" ] ]
                    ]
    in
    toVegaLite
        [ width 540
        , religionData []
        , ps []
        , religionencHighlight []
        , circle []
        ]
```

```elm {l=hidden}
religionData =
    dataFromUrl "https://raw.githubusercontent.com/VithujaThangarajah/data/refs/heads/main/Religion.csv"


religionColours =
    categoricalDomainMap
        [ ( "Female", "rgb(227, 11, 92)" )
        , ( "Male", "rgb(65, 105, 225)" )
        ]


religionencHighlight =
    encoding
        << position X [ pName "Religion", pNominal, pAxis [ axTitle "Religion", axGrid False ] ]
        << position Y [ pName "Employment Percentage", pQuant, pAxis [ axTitle "Employment Percentage", axGrid False ] ]
        << color
            [ mCondition (prParamEmpty "mySelection")
                [ mName "Sex", mScale religionColours ]
                [ mStr "black" ]
            ]
        << opacity
            [ mCondition (prParamEmpty "mySelection")
                [ mNum 1 ]
                [ mNum 0.1 ]
            ]
```

## Visualization 5: By ethnicity and religion, the employment percentages

```elm {l=hidden v interactive highlight=[7,8-18]}
ethnicRadio : Spec
ethnicRadio =
    let
        ps =
            params
                << param "mySelection"
                    [ paBind
                        (ipRadio
                            [ inName "Ethnic Group:"
                            , inOptions
                                [ "Asian/Asian Welsh/Asian British: Bangladeshi"
                                , "Asian/Asian Welsh/Asian British: Chinese"
                                , "Asian/Asian Welsh/Asian British: Indian"
                                , "Asian/Asian Welsh/Asian British: Pakistani"
                                , "Asian/Asian Welsh/Asian British: Other Asian"
                                , "Black/Black Welsh/Black British/Caribbean/African: African background"
                                , "Black/Black Welsh/Black British/Caribbean/African: Caribbean"
                                , "Black/Black Welsh/Black British/Caribbean/African: Other Black"
                                , "Mixed/multiple ethnic group: White and Asian"
                                , "Mixed/multiple ethnic group: White and Black African"
                                , "Mixed/multiple ethnic group: White and Black Caribbean"
                                , "Mixed/multiple ethnic group: Other Mixed"
                                , "Other ethnic group: Arab"
                                , "Other ethnic group: Any other ethnic group"
                                , "White: English/Welsh/Scottish/Northern Irish/British"
                                , "White: Gypsy or Irish Traveller"
                                , "White: Irish"
                                , "White: Roma"
                                , "White: Other White"
                                , "All Adults"
                                , "Asian/Asian Welsh/Asian British: Bangladeshi"
                                , "Asian/Asian Welsh/Asian British: Chinese"
                                , "Asian/Asian Welsh/Asian British: Indian"
                                , "Asian/Asian Welsh/Asian British: Pakistani"
                                , "Asian/Asian Welsh/Asian British: Other Asian"
                                , "Black/Black Welsh/Black British/Caribbean/African: African background"
                                , "Black/Black Welsh/Black British/Caribbean/African: Caribbean"
                                , "Black/Black Welsh/Black British/Caribbean/African: Other Black"
                                , "Mixed/multiple ethnic group: White and Asian"
                                , "Mixed/multiple ethnic group: White and Black African"
                                , "Mixed/multiple ethnic group: White and Black Caribbean"
                                , "Mixed/multiple ethnic group: Other Mixed"
                                , "Other ethnic group: Arab"
                                , "Other ethnic group: Any other ethnic group"
                                , "White: English/Welsh/Scottish/Northern Irish/British"
                                , "White: Gypsy or Irish Traveller"
                                , "White: Irish"
                                , "White: Roma"
                                , "White: Other White"
                                ]
                            ]
                        )
                    , paSelect sePoint [ seFields [ "Ethnic Group" ] ]
                    ]
    in
    toVegaLite
        [ width 540
        , ethnicData []
        , ps []
        , ethnicencHighlight []
        , circle []
        ]
```

```elm {l=hidden}
ethnicData =
    dataFromUrl "https://raw.githubusercontent.com/VithujaThangarajah/data/refs/heads/main/ethnicgroup.csv"


ethnicColours =
    categoricalDomainMap
        [ ( "Asian/Asian Welsh/Asian British: Bangladeshi", "rgb(255, 0, 0)" ) -- Red
        , ( "Asian/Asian Welsh/Asian British: Chinese", "rgb(255, 127, 0)" ) -- Orange
        , ( "Asian/Asian Welsh/Asian British: Indian", "rgb(255, 255, 0)" ) -- Yellow
        , ( "Asian/Asian Welsh/Asian British: Pakistani", "rgb(0, 255, 0)" ) -- Green
        , ( "Asian/Asian Welsh/Asian British: Other Asian", "rgb(0, 0, 255)" ) -- Blue
        , ( "Black/Black Welsh/Black British/Caribbean/African: African background", "rgb(75, 0, 130)" ) -- Indigo
        , ( "Black/Black Welsh/Black British/Caribbean/African: Caribbean", "rgb(238, 130, 238)" ) -- Violet
        , ( "Black/Black Welsh/Black British/Caribbean/African: Other Black", "rgb(255, 0, 255)" ) -- Violet (light)
        , ( "Mixed/multiple ethnic group: White and Asian", "rgb(255, 20, 147)" ) -- Deep Pink
        , ( "Mixed/multiple ethnic group: White and Black African", "rgb(255, 105, 180)" ) -- Hot Pink
        , ( "Mixed/multiple ethnic group: White and Black Caribbean", "rgb(255, 182, 193)" ) -- Light Pink
        , ( "Mixed/multiple ethnic group: Other Mixed", "rgb(255, 165, 0)" ) -- Orange
        , ( "Other ethnic group: Arab", "rgb(255, 99, 71)" ) -- Tomato Red
        , ( "Other ethnic group: Any other ethnic group", "rgb(34, 193, 195)" ) -- Cyan
        , ( "White: English/Welsh/Scottish/Northern Irish/British", "rgb(0, 255, 255)" ) -- Aqua
        , ( "White: Gypsy or Irish Traveller", "rgb(0, 0, 255)" ) -- Blue
        , ( "White: Irish", "rgb(255, 255, 255)" ) -- White
        , ( "White: Roma", "rgb(255, 105, 180)" ) -- Hot Pink
        , ( "White: Other White", "rgb(169, 169, 169)" ) -- Dark Gray
        , ( "All Adults", "rgb(128, 128, 128)" ) -- Gray
        , ( "Asian/Asian Welsh/Asian British: Bangladeshi", "rgb(255, 0, 0)" ) -- Red
        , ( "Asian/Asian Welsh/Asian British: Chinese", "rgb(255, 127, 0)" ) -- Orange
        , ( "Asian/Asian Welsh/Asian British: Indian", "rgb(255, 255, 0)" ) -- Yellow
        , ( "Asian/Asian Welsh/Asian British: Pakistani", "rgb(0, 255, 0)" ) -- Green
        , ( "Asian/Asian Welsh/Asian British: Other Asian", "rgb(0, 0, 255)" ) -- Blue
        , ( "Black/Black Welsh/Black British/Caribbean/African: African background", "rgb(75, 0, 130)" ) -- Indigo
        , ( "Black/Black Welsh/Black British/Caribbean/African: Caribbean", "rgb(238, 130, 238)" ) -- Violet
        , ( "Black/Black Welsh/Black British/Caribbean/African: Other Black", "rgb(255, 0, 255)" ) -- Violet (light)
        , ( "Mixed/multiple ethnic group: White and Asian", "rgb(255, 20, 147)" ) -- Deep Pink
        , ( "Mixed/multiple ethnic group: White and Black African", "rgb(255, 105, 180)" ) -- Hot Pink
        , ( "Mixed/multiple ethnic group: White and Black Caribbean", "rgb(255, 182, 193)" ) -- Light Pink
        , ( "Mixed/multiple ethnic group: Other Mixed", "rgb(255, 165, 0)" ) -- Orange
        , ( "Other ethnic group: Arab", "rgb(255, 99, 71)" ) -- Tomato Red
        , ( "Other ethnic group: Any other ethnic group", "rgb(34, 193, 195)" ) -- Cyan
        , ( "White: English/Welsh/Scottish/Northern Irish/British", "rgb(0, 255, 255)" ) -- Aqua
        , ( "White: Gypsy or Irish Traveller", "rgb(0, 0, 255)" ) -- Blue
        , ( "White: Irish", "rgb(255, 255, 255)" ) -- White
        , ( "White: Roma", "rgb(255, 105, 180)" ) -- Hot Pink
        , ( "White: Other White", "rgb(169, 169, 169)" ) -- Dark Gray
        ]


ethnicencHighlight =
    encoding
        << position X [ pName "Religion", pNominal, pAxis [ axTitle "Religion", axGrid False, axLabelAngle -45 ] ]
        << position Y [ pName "Percentage within ethnic group and religion", pQuant, pAxis [ axTitle "Percentage within ethnic group and religion", axGrid False ] ]
        << color
            [ mCondition (prParamEmpty "mySelection")
                [ mName "Ethnic Group", mScale ethnicColours ]
                [ mStr "black" ]
            ]
        << opacity
            [ mCondition (prParamEmpty "mySelection")
                [ mNum 1 ]
                [ mNum 0.1 ]
            ]
```

## Insight one: Relationship between work history and current employment status in london

Several significant patterns emerge from employment statistics and patterns based on work history:

Worked:

- Former employees have expertise and familiarity with the employment market, which makes them more likely to find employment again. This pattern is most pronounced for people with a steady and ongoing career history. This indicates that prior experience is critical while looking for another job. People with a steady work history in London are more likely to land new jobs since companies emphasise relevant experience.
- Gender breakdowns show males who have worked in the past are often more likely to be employed than women in comparable fields. The truth is women who have previously worked have a higher chance of finding a new job. This is compared to those who have never been employed or out of the workforce for a lengthy period.

Not Worked:

- Re-entering the workforce after a prolonged unemployment period presents difficulties for such individuals. It's possible that these people were unemployed for personal reasons, the economy state, or obstacles to employment. For example, a lack of credentials or a mismatch in skills.
- Nonetheless, a number of these individuals re-enter the workforce through government programs, skills development initiatives, or industry-specific initiatives designed to assist individuals in returning to work following a career break.

Never Worked:

- The highest rates of unemployment or economic inactivity are typically found among never-workers. This group encounters obstacles that prevent them from entering the workforce. These obstacles include insufficient experience, mismatched talents, and a few socioeconomic difficulties.
- Young people and women, particularly those with caregiving obligations, are more likely to fit into this category in the London environment. This contributes to age and gender gaps in the workforce.

Gender:

- Although part-time or lower-paying occupations are more common among women with a work history, they are more likely to return to employment after a break than women who have never worked. For women returning from a caring hiatus, this is particularly true. On the other hand, overall employment rates are marginally higher for men with consistent employment histories.
- Although men's unemployment rates are still high, especially after prolonged periods of unemployment, they exhibit an increased overall employment rate in the "worked" category. One possible explanation for this could be that men are more likely to work in sectors like manufacturing and construction that are susceptible to economic downturns.

Age:

- Young people under 30 who have prior work experience have a higher chance of landing a job. Their ability to adapt to new roles and industries is greater, and entry-level jobs are more likely to be available in London's labour market, especially in the technology, retail, and hospitality sectors.
- Re-entering the workforce presents more challenges for older adults over 50, particularly those who have been unemployed for an extended time. As the labour market changes and technological skills become more important, older workers may find it more difficult to find employment because of age discrimination and changes in job types.

## Insight two: Gender Disparities in Employment Across London Boroughs

- Across London's local authorities, the data shows a continuous pattern of gender discrepancies in employment. This is due to a higher percentage of men working than women in almost every borough.

Proportion of Men and Women Employed:

- Throughout London, the percentage of males in employment continuously exceeds that of women, however this varies by borough.
- The employment disparity between men and women is the narrowest in Wandsworth. This is where 73.2% of men and 66.5% of women work, leaving a 6.7 percentage point gender employment gap.
- However, boroughs like Redbridge and Newham have some of the biggest gender employment inequalities, with the difference being particularly noticeable:
- The employment disparity between men and women in Newham is 14.4 percentage points, with 66.4% of men and 52.0% of women working.
- The disparity is comparable in Redbridge, where 51.8% of women and 65.5% of males are employed, or 13.7 percentage points apart.

Borough-Specific Variations:

- There are regional socioeconomic and cultural factors that could affect employment rates, since there are discrepancies in employment rates between men and women across the several boroughs. When poverty rates are high, single-parent households are prevalent, or education levels are low, women face greater employment obstacles.
- Gender disparities in employment are less pronounced in boroughs with higher population densities or more diverse communities, like Tower Hamlets and Hackney, than in outer boroughs, like Bexley and Bromley. Both sexes may have more opportunities in cities with a vibrant and diverse labour market.

Unemployment Rates by Gender:

- London's unemployed women are significantly higher than unemployed men. These differences are especially noticeable in certain boroughs, such as Newham.
- At 5.8%, Newham has the highest rate of female unemployment in all of London. At 6.7%, the borough also has the second-highest male unemployment rate. This implies that higher unemployment rates for both sexes in these boroughs are caused by elements such as the economic downturn, a lack of employment prospects, and restricted access to vocational training.
- Richmond upon Thames, Sutton, and Bromley have lower unemployment rates for both sexes. This may indicate that these places have higher-paying jobs or are less affected by economic downturns.

In London, there are significant differences between the employment outcomes and opportunities for women and men due to the complex interaction between socio-economic factors and gender dynamics.

## Insight three: The influence of age and gender on employment rates

The disparities in employment rates in England and Wales by gender and age group are clearly visible in this visualization. It confirms a long-standing pattern: men are more likely than women to be employed, and the discrepancy grows as people get older. Male employment (81.6%) is substantially higher than female employment (76.6%) in the 25–29 age group, for instance. With 85.2% of men and 75.5% of women working in the 30–34 age range, this disparity is even more noticeable.

Men and women are less likely to have different employment situations than younger individuals, especially those between 16 and 19. Among young adults in this age group, many are still studying rather than working, making the employment rate for women higher (24.5%) than for men (23.7%), though both rates are low overall. People in older age groups are more likely to be employed, reaching a peak at 80.8% between 40 and 44. People experience the greatest losses in employment after 60 years of age, for both sexes.

Males continuously have greater employment rates than females across almost all age categories, indicating that gender differences in work continue into adulthood. The years spent raising children frequently coincide with unpaid caregiving obligations, which could explain this discrepancy. Men are therefore more likely to work full-time throughout their lives, whereas women are more likely to take professional pauses or choose part-time employment.

## Insight four: Employment Disparities Across Demographic Categories

Understanding the differences in employment by age, gender, ethnicity, religion, and geography is made evident by examining employment statistics from all five visualisations. By highlighting the interdependence of demographic characteristics and their impact on job outcomes, these visualisations enable us to identify patterns, gaps, and areas of concern.

Gender Disparities:

- Visualisations 1 and 4 show gender-based differences in employment rates across age groups and religions. For example, compared to women, men's employment rates are consistently higher. All religious groupings exhibit gender discrepancies, with male employment rates being substantially higher, particularly within certain religions, as illustrated in Visualisation 4. Blue men perform better than pink women, as shown by the colour-coded bar charts. These results support more general societal patterns about gender disparities in the workplace (Tufte, 2001).

Ethnic and Religious Disparities:

- A more sophisticated picture of job differences across these categories is offered by Visualisation 5, which focuses on religion and ethnicity. Clearly, some religious groups are disproportionately disadvantaged, and some ethnic groups have lower employment rates than others. This visualisation enhances understanding of which populations have the most employment issues by enabling users to isolate and compare ethnic groupings with interactive radio buttons. As a result, minorities exhibit varied degrees of job gaps, for instance, underscoring structural injustices. Visualisation 4 reflects these patterns, where religious differences exacerbate job difficulties.

Age-Based Trends:

- Employment rates are impacted by age, as seen in Visualisation 1. Middle-aged groups (25–50 years old) have the largest employment percentages, whilst younger and older populations have far lower employment rates. This realisation implies that while older people cope with issues like age discrimination or ability disparities, younger people can encounter obstacles to entering the workforce. It is evident that the intersection of age with other demographic characteristics (such as gender or ethnicity) exacerbates employment disparities when this is linked to visualisations 4 (gender) and 5 (ethnicity/religion).

Spatial Patterns in Employment:

- London boroughs' employment rates vary geographically, as seen in Visualisation 3 (choropleth maps). Employment rates are lower in some areas. This could reflect regional economic issues including systemic inequality, a lack of opportunities, or low educational attainment. But when you overlay the results from Visualisations 4 and 5 (religion, gender, and ethnicity), you can see how spatial differences interact with demographic categories. For example, women and ethnic minorities may experience additional difficulties in some boroughs, which could exacerbate employment disparities. Added overlays to choropleth maps with demographic-specific filters would enhance clarity and enable users to delve deeper into these links.

Conclusion:

- When all five visualisations are analysed together, it becomes clear that there are many different factors that contribute to job gaps, including age, gender, location, ethnicity, and religion. Instead of acting independently, these variables combine to produce multilayer inequality. Across all age groups and religions, gender disparities still exist, and employment differences are made worse by ethnicity and religion. These disparities are exacerbated by localised economic difficulties, according to geographic trends. These findings highlight the necessity of focused policy initiatives that address the wider confluence of age, gender, ethnicity, and geography rather than simply one demographic feature, as recommended by Kirk (2016) and Yau (2011).


## Visualization 1: Employment and Work History

Choropleth maps are suitable for displaying job and labour history data for several London regions. This is because they enable visual spatial data comparison across several geographical regions. To improve clarity and allow for a direct comparison of these categories, three distinct maps are used for "Worked," "Not Worked," and "Never Worked." Small multiples, as noted by Munzner (2014), enable viewers to compare multiple visual representations of comparable data structures while preserving coherence, particularly when dealing with huge, complicated datasets. The colour gradient of each map red for unemployment and blue for employment makes it easier to spot patterns and differentiate between job status categories.

Using red for unemployment and blue for employment is a logical colour selection that complies with accepted colour schemes in data visualisation. Blue is associated with stability and serenity, which is appropriate for employment statistics. In contrast, red denotes issues or unfavourable circumstances, which is appropriate for unemployment data. Effective colour selection should make it easier to understand the data by establishing intuitive connections with its meaning, according to Stone (2006). Making information instantly recognisable and lowering cognitive load are two more best practices followed in this strategy of employing colour for categorical separation.

Additionally, the choropleth structure makes it easier to visually examine employment patterns in various regions. Tufte (2001) supports the use of clear, uncomplicated images that eliminate extraneous distractions and enable viewers to recognise patterns. The most significant patterns in the data are easily discernible because of the use of choropleth maps. These maps provide users to examine the geographical distribution of employment and work history throughout London.

In conclusion, by adhering to best practices in colour selection, layout, and data display, the use of choropleth maps with blue and red colour schemes allows for a clear, intuitive representation of employment data spatial distribution. This preserves clarity and insight while enabling consumers to compare across locations.


## Visualization 2: Employment and Unemployment Rates by Gender

A targeted comparison of employment, unemployment, and gender-based disparities in job status is made possible using distinct bar charts for each of these metrics. Separating related data into several targeted visualisations aids users in better comprehending the subtleties of the data (Equaler et al., 2011). These categories are divided into separate charts so that the viewer may quickly and clearly see trends and variations in job outcomes.

To establish a clear distinction between genders, the colour red is used for women's employment and blue for men's. Kirk (2016) emphasises that to effectively visualise data, contrasting colours must be used so that users can quickly recognise the category of interest. The customary recognition of red and blue as gendered colours also improves the user's intuitive comprehension of the data. This is particularly true when analysing gender-based differences in unemployment and employment.

To help viewers better comprehend the differences between men and women in these two statuses, employment and unemployment data are presented separately in this visualisation. Munzner (2014) discusses how this enables a more accurate comparison of gender-based employment rates. The possibility of cognitive overload that could result from comparing all the data in a single chart is reduced when these categories are shown independently. By offering a simple and approachable way to visualise the data, bar charts are also in line with accepted best practices for displaying categorical comparisons.

The color-coded bar charts provide a clear, easily interpretable visual representation of employment and unemployment data by gender, facilitating focused comparisons that are aligned with good visualization practices.


## Visualization 3: Age and Employment Status by Gender

Since it allows for direct comparison between different demographic categories in a single, condensed visual representation, the stacked bar chart is a useful design option for displaying employment rates by gender and age in London. Munzner (2014) asserts that a stacked bar chart is suitable for illustrating the intricate interactions between age and gender in this dataset since it is clear when comparing relative numbers across categories. While the segments of the bars illustrate the gender breakdown within each age group, the X-axis depicts the various age groups, and the Y-axis displays the employment %. A contrast between purple and light purple provides instant visual differentiation, making it easy to understand the trends for men and women.

Based on the data, this methodology provides a means of evaluating gender-based differences in employment rates. Meulemans et al. (2017) emphasise the value of clear and concise visualisations when working with demographic data. This is particularly true when several factors, such as age and sex, must be considered at the same time. Viewers may easily spot gender variances between age groups thanks to the stacked bar chart. Understanding the unique patterns of labour market involvement across genders depends on how simple it is to compare males and females in each age group.

Additionally, the chart's colour scheme is based on data visualisation best practices to enhance perceptual comprehension. Stone (2006) highlights how crucial it is to use colour in data visualisations to improve readability and draw attention to significant trends. The stacked bar chart's unique colour scheme makes it easier to distinguish between the sexes. It also ensures that the viewer can concentrate on significant details without being overloaded with data. The design of the stacked bar chart also complies with Tufte (2001) guidelines, who promote the use of straightforward, understandable graphics to effectively communicate information to maximise the data to ink ratio. The relationship between age, gender, and employment is clearly displayed in this chart. This reduces superfluous elements and is the most effective option for presenting facts in an understandable way.

Exploring demographic differences in employment with this approach also connects to the larger topic of visualisation analysis. Yau (2011) says when the design facilitates intuitive pattern discovery, these visualisations can reveal significant insights from unprocessed data. The stacked bar chart, which focuses on the gender-based employment gap, makes it simple for viewers to see how age affects this gap and how different age groups encounter varied employment levels between males and females. These sources collectively bolster the design decision that a stacked bar chart is the most effective method of displaying intricate statistical data. This guarantees viewers' comprehension and useful insights.


## Visualization 4: By religion and gender, the employment percentages

Using colour coding and interactive elements, Visualisation 4's design makes it easy to see employment rates by gender and religion. This visualization's radio buttons let viewers filter data by gender is one of its main features. By enabling users to evaluate the differences in employment rates between various religions and concentrate on a particular demographic category (either male or female), this interactivity is crucial for streamlining data research. This filtering method improves the user experience, allowing personalised data display while keeping the correlation between employment percentages, gender, and religion in sharp focus.

Colour coding remains a significant design choice. Male employment data is shown in blue (rgb(65, 105, 225)), whereas female employment data is shown in a bright pink tone (rgb(227, 11, 92)). Viewers can better understand the gender breakdown in the data thanks to this colour scheme's intuitiveness and popular gender associations. Additionally, the contrast produced by these colours makes the data easily recognisable and visually appealing. The opacity is changed, and the chart is displayed in black when no gender is chosen. This indicates that the data is not gender-filtered and that all data is shown in a neutral baseline view.

Visualisation 4 effectively illustrates employment numbers for various religious groups through a bar chart. Bar charts make it simple to compare various groups and are an excellent tool for displaying categorical data, such as religion. The visualisation displays the data in a clear and understandable way by aligning the X-axis with religion and the Y-axis with employment %. This makes it possible for viewers to quickly compare the employment statistics of different religious groups. This is particularly helpful when spotting trends or discrepancies in employment across gender and religion.

Furthermore, by minimising visual clutter when the user chooses a certain gender, the usage of a dynamic opacity setting based on user decision complies with top data visualisation principles. It's critical to remove any extraneous distractions so that consumers can concentrate on pertinent data, as per data visualisation design principles (Tufte, 2001). To reduce cognitive overload and preserve clarity, the design modifies the opacity of unselected data. In addition to making the chart easier to read, this design choice guarantees that the user can concentrate on the chosen subset of data with ease. This improves the visualization's overall usefulness.


## Visualization 5: By ethnicity and religion, the employment percentages

Colour-coding and interactive elements are used in Visualisation 5 to demonstrate employment rates by ethnicity and religion. Visualization offers a simple way to distinguish various demographic groups by colour coding. Colour is a crucial element in data visualisation, according to Stone (2006), since it enables viewers to easily connect multiple values or categories with colours, enhancing their comprehension of complex data. Viewers may compare employment patterns across various groups thanks to different hues for each ethnicity and religion. This method not only helps identify trends but also helps gain key insights. This is consistent with Tufte's (2001) idea that reducing visual clutter improves data interpretation.

Users can filter data by ethnicity using the interactive radio button element, which significantly enhances visualisation. By modifying the display according to their individual interests, interactive aspects allow users to examine the material more thoroughly and intimately. This is explained by Munzner (2014). In this instance, the visualisation becomes more dynamic and captivating since viewers may concentrate on ethnic groups and analyse how their employment rates differ from those of other groups. Kirk (2016) highlights personalised filters are an effective means of offering customised insights while preserving visual representation's efficacy and clarity.

In addition, Visualisation 5's design uses the idea of "small multiples," a method that Meulemans et al. (2017) emphasises, and which displays the same data in several panels for simpler comparison. Users can easily spot variations and patterns between each ethnic group without being distracted by a crowded chart. This is because each group is shown in a separate view. This technique of segmenting the data into smaller, equivalent units makes it easier to analyse job differences by religion and ethnicity in greater detail. Small multiples are compact, which prevents cognitive overload and makes analysis more targeted and effective.

Lastly, Gleicher et al. (2011) describe how visualization's application of visual comparison principles improves overall effectiveness. Through interactive filtering and colour-coded radio buttons, the user can compare employment rates for different demographic groups side-by-side. A more comparative approach to data analysis is encouraged by the design's easy category switching. This is especially helpful for examining delicate subjects like employment gaps by religion and race. These guidelines support standard practices in information visualisation design while creating an interesting and perceptive user experience.


## Graphical integrity

The five visualisations use a uniform method of data display with a focus on graphical integrity, clarity, and usability. They cover a range of demographic characteristics, including gender, ethnicity, age, and location. Tufte (2001) underlines that the visualisations guarantee an accurate and truthful portrayal of the data through appropriate scaling, proportionality, and clear axis labels. When displaying employment percentages by age, gender, and religion, the stacked bar charts in Visualisations 1 and 4 adopt consistent scales and axis labelling. This allows for a reliable comparison between various groups. The stacked bar charts, for instance, allow users to compare employment percentages across demographic groups like age, gender, and religious affiliation easily because the Y-axis is uniformly scaled across all visualisations. A possible drawback of these visualisations, meanwhile, is data aggregation, which ignores individual-level variances in favour of group-level patterns. This aggregate may mask significant variances or outliers that could offer a more profound interpretation of the data, as Yau (2011) points out. The visualisations may enable users to dig deeper into these nuances and develop a more complex comprehension of the data. This is done by integrating more detailed disaggregation elements.


## Clarity

Clarity is given priority in the design of these visualisations. This is accomplished by using small multiples for comparison analysis, logical layouts, and appealing colour schemes. To differentiate between various demographic groupings, Visualisations 4 and 5 apply colour coding. For instance, employment rates for males and females are represented by blue and pink, respectively. In addition, the categories of ethnicity and religion are colour-coded in distinct shades. This is in line with Stone (2006)'s suggestions to reduce ambiguity and enhance comprehension. Additionally, Visualisations 4 and 5 improve clarity by dividing complex data into smaller, more manageable chunks through tiny multiples. Using this method, Meulemans et al. (2017) describe how users can concentrate on specific data points or trends without being overwhelmed by data. One drawback of the choropleth maps is that they don't give viewers unfamiliar with the regions—like London boroughs—the contextual information they need. As Kirk (2016) suggests, adding geographical overlays or labels could improve the interpretability of spatial data. This would give users a greater understanding of job discrepancies' geographical context.


## Usability

By giving consumers the flexibility to select data and concentrate on demographic groups, the interactive components included in the visualisations improve usability. For a more individualised data exploration experience, users can switch between gender and ethnicity using the radio buttons in Visualisations 4 and 5, respectively. Such interactive elements enable a deeper interaction with the data, which makes the visualisations more dynamic and approachable, as Munzner (2014) points out. Viewers can focus on categories, such as male or female employment statistics in Visualisation 4 or specific ethnic groups in Visualisation 5, thanks to filtering choices. This makes the experience more meaningful and tailored. The possibility for more research is, however, constrained by the static character of some visualisations, especially the choropleth maps. It would greatly improve the user experience to include tools like zooming, hover tooltips, or click-to-drill capabilities, which would allow for more detailed investigation of demographic and spatial patterns. As Kirk (2016) suggests, users could interact with choropleth maps to display more data points or filter regions.


## Improvements

Future versions might incorporate more capabilities for data disaggregation and improved interaction to further enrich the visualisations. As suggested by Yau (2011), users might find it useful to switch between aggregated and disaggregated views. This would enable a more thorough examination of trends and individual-level data. This would allow for a more sophisticated interpretation of the data by giving a more complete picture of job gaps across various demographic categories. Furthermore, since geographical context is essential for properly analysing the data, adding geographical labels or overlays to choropleth maps would improve spatial comprehension (Kirk, 2016). Lastly, the visualisations would become even more intuitive and informative with the addition of features like side-by-side comparisons or interactive filters for areas or subgroups. According to Tufte (2001), Kirk (2016), and Munzner (2014), these enhancements would give users more freedom in examining and analysing the data while maintaining the clarity and simplicity of the current design.


## References

- Gleicher, M., Albers, D., Walker, R., Jusufi, I., Hansen, C.D., and Roberts, J.C. (2011). Visual comparison for information visualization. Information Visualization.
- Kirk, A. (2016). Data visualisation: a handbook for data driven design. Sage.
- Munzner, T. (2014). Visualization Analysis and Design. AK Peters/CRC Press.
- Meulemans, W., Dykes, J., Slingsby, A., Turkay, C., & Wood, J. (2017). Small Multiples with Gaps. IEEE Transactions on Visualization and Computer Graphics.
- Satyanarayan, A., Moritz, D., Wongsuphasawat, K., & Heer, J. (2017). Vega-lite: A grammar of interactive graphics. IEEE Transactions on Visualization and Computer Graphics.
- Stone, M. (2006). Choosing Colors for Data Visualization. Perceptual Edge.
- Tufte, E. R. (2001). The Visual Display of Quantitative Information (Vol. 2). Cheshire, CT: Graphics Press.
- Yau, N. (2011). Visualize This: The FlowingData Guide to Design, Visualization, and Statistics. John Wiley & Sons.


```elm {l=hidden}
employedTable : Table
employedTable =
    """
borough,percent
Hartlepool,50.5
Middlesbrough,49.5
Redcar and Cleveland,49.5
Stockton-on-Tees,54.7
Darlington,56.0
Halton,56.5
Warrington,59.7
Blackburn with Darwen,53.2
Blackpool,51.0
Kingston upon Hull City of,55.1
East Riding of Yorkshire,54.2
North East Lincolnshire,54.0
North Lincolnshire,54.7
York,55.9
Derby,55.5
Leicester,53.2
Rutland,54.8
Nottingham,48.8
Herefordshire, County of,56.3
Telford and Wrekin,57.5
Stoke-on-Trent,54.7
Bath and North East Somerset,55.6
Bristol City of,61.2
North Somerset,57.5
South Gloucestershire,62.2
Plymouth,55.4
Torbay,50.4
Swindon,63.0
Peterborough,60.3
Luton,57.2
Southend-on-Sea,56.9
Thurrock,63.0
Medway,59.5
Bracknell Forest,66.4
West Berkshire,63.0
Reading,62.9
Slough,62.0
Windsor and Maidenhead,61.7
Wokingham,63.5
Milton Keynes,63.8
Brighton and Hove,59.2
Portsmouth,57.2
Southampton,57.9
Isle of Wight,48.4
County Durham,51.2
Cheshire East,58.4
Cheshire West and Chester,57.2
Shropshire,56.0
Cornwall,53.3
Isles of Scilly,64.4
Wiltshire,60.2
Bedford,61.3
Central Bedfordshire,63.8
Northumberland,52.4
Bournemouth Christchurch and Poole,55.5
Dorset,52.1
Buckinghamshire,61.6
North Northamptonshire,61.8
West Northamptonshire,62.9
Cambridge,56.6
East Cambridgeshire,62.6
Fenland,55.5
Huntingdonshire,61.6
South Cambridgeshire,63.1
Allerdale,54.9
Barrow-in-Furness,55.8
Carlisle,58.0
Copeland,54.6
Eden,58.6
South Lakeland,55.9
Amber Valley,57.3
Bolsover,57.5
Chesterfield,55.3
Derbyshire Dales,54.4
Erewash,58.8
High Peak,59.0
North East Derbyshire,54.6
South Derbyshire,62.0
East Devon,52.4
Exeter,53.6
Mid Devon,59.1
North Devon,55.0
South Hams,54.1
Teignbridge,54.8
Torridge,52.2
West Devon,52.5
Eastbourne,51.8
Hastings,54.7
Lewes,53.5
Rother,47.7
Wealden,55.8
Basildon,59.9
Braintree,60.3
Brentwood,60.3
Castle Point,55.1
Chelmsford,60.8
Colchester,58.9
Epping Forest,59.9
Harlow,62.9
Maldon,56.4
Rochford,57.5
Tendring,46.7
Uttlesford,62.2
Cheltenham,60.1
Cotswold,57.8
Forest of Dean,54.7
Gloucester,61.4
Stroud,59.8
Tewkesbury,60.1
Basingstoke and Deane,64.7
East Hampshire,59.5
Eastleigh,63.0
Fareham,58.2
Gosport,57.3
Hart,62.8
Havant,54.8
New Forest,52.9
Rushmoor,65.6
Test Valley,61.7
Winchester,57.8
Broxbourne,60.8
Dacorum,62.4
Hertsmere,61.2
North Hertfordshire,62.3
Three Rivers,62.6
Watford,65.3
Ashford,59.6
Canterbury,49.8
Dartford,64.9
Dover,53.0
Gravesham,58.8
Maidstone,60.8
Sevenoaks,58.4
Folkestone and Hythe,52.2
Swale,56.7
Thanet,51.3
Tonbridge and Malling,61.1
Tunbridge Wells,60.5
Burnley,53.2
Chorley,59.4
Fylde,51.6
Hyndburn,54.3
Lancaster,52.2
Pendle,52.7
Preston,56.2
Ribble Valley,59.0
Rossendale,57.8
South Ribble,59.8
West Lancashire,54.5
Wyre,50.7
Blaby,61.5
Charnwood,55.4
Harborough,60.6
Hinckley and Bosworth,59.5
Melton,59.3
North West Leicestershire,60.3
Oadby and Wigston,55.0
Boston,58.4
East Lindsey,45.8
Lincoln,53.7
North Kesteven,58.0
South Holland,57.3
South Kesteven,57.8
West Lindsey,53.3
Breckland,55.7
Broadland,56.6
Great Yarmouth,49.7
King's Lynn and West Norfolk,53.7
North Norfolk,46.7
Norwich,54.4
South Norfolk,57.8
Craven,56.2
Hambleton,56.4
Harrogate,58.9
Richmondshire,59.1
Ryedale,56.0
Scarborough,50.5
Selby,61.1
Ashfield,56.8
Bassetlaw,56.4
Broxtowe,56.1
Gedling,58.2
Mansfield,57.0
Newark and Sherwood,56.0
Rushcliffe,58.3
Cherwell,65.1
Oxford,53.6
South Oxfordshire,62.9
Vale of White Horse,63.0
West Oxfordshire,62.8
Mendip,58.0
Sedgemoor,57.0
South Somerset,56.3
Cannock Chase,59.3
East Staffordshire,59.0
Lichfield,57.0
Newcastle-under-Lyme,54.9
South Staffordshire,54.3
Stafford,57.9
Staffordshire Moorlands,55.7
Tamworth,59.8
Babergh,55.5
Ipswich,59.9
Mid Suffolk,58.0
Elmbridge,61.6
Epsom and Ewell,61.8
Guildford,58.5
Mole Valley,58.5
Reigate and Banstead,62.5
Runnymede,58.4
Spelthorne,61.8
Surrey Heath,61.5
Tandridge,60.1
Waverley,58.6
Woking,63.6
North Warwickshire,58.9
Nuneaton and Bedworth,58.1
Rugby,63.0
Stratford-on-Avon,58.8
Warwick,59.0
Adur,56.3
Arun,52.4
Chichester,53.8
Crawley,62.9
Horsham,59.8
Mid Sussex,62.1
Worthing,58.0
Bromsgrove,58.3
Malvern Hills,53.3
Redditch,60.8
Worcester,60.3
Wychavon,57.5
Wyre Forest,54.1
St Albans,62.8
Welwyn Hatfield,59.6
East Hertfordshire,63.2
Stevenage,63.8
East Suffolk,51.5
West Suffolk,60.5
Somerset West and Taunton,55.8
Bolton,54.0
Bury,57.1
Manchester,53.5
Oldham,52.5
Rochdale,54.0
Salford,58.8
Stockport,58.5
Tameside,57.1
Trafford,59.9
Wigan,57.5
Knowsley,55.9
Liverpool,51.1
St. Helens,54.9
Sefton,53.6
Wirral,53.2
Barnsley,55.9
Doncaster,54.9
Rotherham,54.6
Sheffield,53.0
Newcastle upon Tyne,50.3
North Tyneside,56.7
South Tyneside,51.5
Sunderland,51.8
Birmingham,49.7
Coventry,54.2
Dudley,55.1
Sandwell,53.0
Solihull,56.1
Walsall,53.1
Wolverhampton,53.4
Bradford,52.5
Calderdale,56.0
Kirklees,54.9
Leeds,56.8
Wakefield,57.3
Gateshead,54.1
City of London,66.9
Barking and Dagenham,58.5
Barnet,60.2
Bexley,60.2
Brent,58.5
Bromley,61.2
Camden,57.6
Croydon,61.0
Ealing,59.3
Enfield,55.9
Greenwich,62.2
Hackney,64.1
Hammersmith and Fulham,63.4
Haringey,62.2
Harrow,59.6
Havering,59.5
Hillingdon,59.0
Hounslow,60.7
Islington,62.6
Kensington and Chelsea,56.4
Kingston upon Thames,62.1
Lambeth,68.0
Lewisham,64.2
Merton,64.9
Newham,59.2
Redbridge,58.5
Richmond upon Thames,62.6
Southwark,65.2
Sutton,62.9
Tower Hamlets,61.4
Waltham Forest,63.2
Wandsworth,69.6
Westminster,57.3
Isle of Anglesey,51.1
Gwynedd,52.9
Conwy,50.6
Denbighshire,51.8
Flintshire,57.9
Wrexham,56.0
Ceredigion,49.1
Pembrokeshire,51.5
Carmarthenshire,52.7
Swansea,50.9
Neath Port Talbot,50.5
Bridgend,53.6
Vale of Glamorgan,55.8
Cardiff,54.1
Rhondda Cynon Taf,53.3
Caerphilly,54.0
Blaenau Gwent,52.2
Torfaen,54.1
Monmouthshire,54.7
Newport,57.1
Powys,54.9
Merthyr Tydfil,53.8"""
        |> fromCSV
```

```elm {l=hidden}
unemployedTable : Table
unemployedTable =
    """
borough,percent
Hartlepool,4
Middlesbrough,5.1
Redcar and Cleveland,3.6
Stockton-on-Tees,3.7
Darlington,3.4
Halton,3.2
Warrington,2.8
Blackburn with Darwen,3.8
Blackpool,4.3
"Kingston upon Hull, City of",4.8
East Riding of Yorkshire,2.3
North East Lincolnshire,3.3
North Lincolnshire,3
York,2.9
Derby,3.9
Leicester,5
Rutland,1.9
Nottingham,5.3
"Herefordshire, County of",2.3
Telford and Wrekin,3.3
Stoke-on-Trent,3.6
Bath and North East Somerset,2.9
"Bristol, City of",3.6
North Somerset,2.2
South Gloucestershire,2.5
Plymouth,3.3
Torbay,2.9
Swindon,3.2
Peterborough,3.7
Luton,5.3
Southend-on-Sea,3.7
Thurrock,3.8
Medway,3.6
Bracknell Forest,3
West Berkshire,2.4
Reading,4.1
Slough,5.1
Windsor and Maidenhead,3.1
Wokingham,2.7
Milton Keynes,3.7
Brighton and Hove,4.6
Portsmouth,4.4
Southampton,4.1
Isle of Wight,2.9
County Durham,3.1
Cheshire East,2.4
Cheshire West and Chester,2.8
Shropshire,2.4
Cornwall,2.6
Isles of Scilly,0.9
Wiltshire,2.3
Bedford,3.3
Central Bedfordshire,2.6
Northumberland,2.9
"Bournemouth, Christchurch and Poole",3.3
Dorset,2.1
Buckinghamshire,2.8
North Northamptonshire,2.7
West Northamptonshire,3
Cambridge,2.9
East Cambridgeshire,2.1
Fenland,2.7
Huntingdonshire,2.3
South Cambridgeshire,2.1
Allerdale,2.2
Barrow-in-Furness,2.7
Carlisle,2.5
Copeland,2.5
Eden,1.8
South Lakeland,1.7
Amber Valley,2.3
Bolsover,2.3
Chesterfield,2.7
Derbyshire Dales,1.9
Erewash,2.7
High Peak,2.5
North East Derbyshire,2.2
South Derbyshire,2.3
East Devon,1.9
Exeter,3.2
Mid Devon,2
North Devon,2.4
South Hams,1.9
Teignbridge,2.1
Torridge,2.4
West Devon,1.9
Eastbourne,3.5
Hastings,3.8
Lewes,2.8
Rother,2.5
Wealden,2.2
Basildon,3.3
Braintree,2.8
Brentwood,2.6
Castle Point,2.6
Chelmsford,2.7
Colchester,3.5
Epping Forest,3.1
Harlow,3.9
Maldon,2.4
Rochford,2.6
Tendring,3.1
Uttlesford,2.4
Cheltenham,2.8
Cotswold,2.1
Forest of Dean,2.4
Gloucester,3.1
Stroud,2.1
Tewkesbury,2.2
Basingstoke and Deane,2.8
East Hampshire,2.3
Eastleigh,2.4
Fareham,2.2
Gosport,3.1
Hart,2.4
Havant,2.9
New Forest,2.1
Rushmoor,3.1
Test Valley,2.2
Winchester,2.6
Broxbourne,3.4
Dacorum,2.9
Hertsmere,3.2
North Hertfordshire,2.6
Three Rivers,2.9
Watford,3.8
Ashford,2.9
Canterbury,3.9
Dartford,3.3
Dover,3.2
Gravesham,3.8
Maidstone,2.7
Sevenoaks,2.4
Folkestone and Hythe,3.2
Swale,3.2
Thanet,3.6
Tonbridge and Malling,2.4
Tunbridge Wells,2.6
Burnley,3.6
Chorley,2.4
Fylde,2.3
Hyndburn,3.4
Lancaster,2.8
Pendle,3.2
Preston,3.8
Ribble Valley,1.7
Rossendale,2.7
South Ribble,2.2
West Lancashire,3
Wyre,2.6
Blaby,2.5
Charnwood,2.9
Harborough,1.9
Hinckley and Bosworth,2.2
Melton,2.1
North West Leicestershire,2.4
Oadby and Wigston,3.1
Boston,2.8
East Lindsey,2.9
Lincoln,4.5
North Kesteven,2.1
South Holland,2.5
South Kesteven,2.5
West Lindsey,2.6
Breckland,2.4
Broadland,2
Great Yarmouth,4
King's Lynn and West Norfolk,2.5
North Norfolk,2.2
Norwich,4.4
South Norfolk,2
Craven,1.7
Hambleton,1.7
Harrogate,1.9
Richmondshire,2
Ryedale,1.8
Scarborough,2.6
Selby,2.3
Ashfield,3
Bassetlaw,2.4
Broxtowe,2.7
Gedling,2.6
Mansfield,2.8
Newark and Sherwood,2.4
Rushcliffe,2.2
Cherwell,2.5
Oxford,3.2
South Oxfordshire,2.2
Vale of White Horse,2.1
West Oxfordshire,2
Mendip,2.3
Sedgemoor,2.4
South Somerset,2.2
Cannock Chase,3
East Staffordshire,2.8
Lichfield,2.4
Newcastle-under-Lyme,2.9
South Staffordshire,2.4
Stafford,2.3
Staffordshire Moorlands,2
Tamworth,3.2
Babergh,2.4
Ipswich,3.7
Mid Suffolk,2
Elmbridge,2.8
Epsom and Ewell,2.8
Guildford,3.1
Mole Valley,2.3
Reigate and Banstead,2.8
Runnymede,3.6
Spelthorne,3.6
Surrey Heath,2.8
Tandridge,2.6
Waverley,2.3
Woking,3
North Warwickshire,2.7
Nuneaton and Bedworth,3.2
Rugby,2.5
Stratford-on-Avon,2
Warwick,2.8
Adur,2.7
Arun,2.6
Chichester,2.6
Crawley,5
Horsham,2.4
Mid Sussex,2.4
Worthing,3
Bromsgrove,2.5
Malvern Hills,2.3
Redditch,3.1
Worcester,3.5
Wychavon,2.2
Wyre Forest,2.9
St Albans,2.6
Welwyn Hatfield,3.9
East Hertfordshire,2.5
Stevenage,3.3
East Suffolk,2.6
West Suffolk,2.4
Somerset West and Taunton,2.2
Bolton,3.9
Bury,3.2
Manchester,5.7
Oldham,4.1
Rochdale,3.8
Salford,4.7
Stockport,2.8
Tameside,3.5
Trafford,3
Wigan,3.1
Knowsley,3.5
Liverpool,4.8
St. Helens,3.1
Sefton,3
Wirral,2.9
Barnsley,2.9
Doncaster,3.3
Rotherham,3.2
Sheffield,4
Newcastle upon Tyne,4.8
North Tyneside,3.3
South Tyneside,4.4
Sunderland,3.9
Birmingham,5.9
Coventry,4.9
Dudley,3.8
Sandwell,5.3
Solihull,3.2
Walsall,4.5
Wolverhampton,5.6
Bradford,4.3
Calderdale,3.2
Kirklees,3.5
Leeds,4.1
Wakefield,3
Gateshead,3.7
City of London,4.1
Barking and Dagenham,5.6
Barnet,4.4
Bexley,3.8
Brent,5.4
Bromley,3.4
Camden,5.1
Croydon,4.7
Ealing,5.2
Enfield,4.7
Greenwich,5.3
Hackney,5.5
Hammersmith and Fulham,5
Haringey,5.5
Harrow,4.1
Havering,3.6
Hillingdon,4.8
Hounslow,5.2
Islington,5.4
Kensington and Chelsea,4.6
Kingston upon Thames,3.9
Lambeth,5.3
Lewisham,5.7
Merton,4
Newham,6.3
Redbridge,4.5
Richmond upon Thames,3.4
Southwark,5.6
Sutton,3.4
Tower Hamlets,6
Waltham Forest,5.2
Wandsworth,4
Westminster,4.8
Isle of Anglesey,2.9
Gwynedd,3.2
Conwy,3
Denbighshire,2.8
Flintshire,2.8
Wrexham,2.9
Ceredigion,3
Pembrokeshire,2.7
Carmarthenshire,2.5
Swansea,3.4
Neath Port Talbot,2.8
Bridgend,2.9
Vale of Glamorgan,2.8
Cardiff,4.1
Rhondda Cynon Taf,3.2
Caerphilly,3.1
Blaenau Gwent,3.4
Torfaen,3
Monmouthshire,2.2
Newport,3.5
Powys,2.1
Merthyr Tydfil,3.2"""
        |> fromCSV
```
