---
id: litvis

narrative-schemas:
  - ../narrative-schemas/teaching.yml

elm:
  dependencies:
    gicentre/elm-vegalite: latest
    gicentre/elm-vega: latest
    gicentre/tidy: latest
---

@import "../css/datavis.less"

```elm {l=hidden}
import Bitwise
import Dict
import Set
import Tidy exposing (..)
import Vega as V
import VegaLite exposing (..)
```

<!-- Everything above this line should probably be left untouched. -->

# Session 9: Representing Networks

## Table of Contents

1. [Introduction](#1-introduction)
2. [Node-link Representation](#2-node-link-representation)
3. [Sankey Diagrams](#3-sankey-diagrams)
4. [Matrix View of Networks](#4-matrix-view-of-networks)
5. [Conclusions](#5-conclusions)
6. [Recommended Reading](#6-recommended-reading)
7. [Practical Exercises](#7-practical-exercises)

{(aim|}

This session is designed to show how networks can be represented graphically. It considers the characteristics of data that make them amenable to network representation and some of the alternative network representations available.

By the end of this session you should be able to:

- critique existing network visualizations identifying their strengths and weaknesses
- show a node-link network in a sketch
- represent a network as a matrix

{|aim)}

---

## 1. Introduction

Connections between items of data can be important to understand, but are often poorly expressed using traditional statistical data graphics (bar charts, scatterplots etc.). This session considers some of the ways in which these connections may be visualized in order to understand their relationships and answer questions about the networks they form. Examples of the types of research questions that we may ask of networks include

- To what extent are some authors more important than others in maintaining a co-authorship network?
- How do the connections between friends in a social network differ from those we might expect in a random network?
- Where are the vulnerabilities in the telecommunication network of a city?
- How can congestion patterns of morning commuters identify where it is particularly important to maintain good traffic flow?
- How do 'patient pathways' of referrals and consultations differ for people who end up having hospital treatment?
- What are the similarities and differences between the use of language in a collection of texts?
- What are the transitions between the states of a Finite State Machine?

When characterising networks of data, it is common to identify two important elements. **Nodes** represent the individual data items in a network, such as authors, words, people or electricity substations. Links or **edges** represent the connections between nodes such as co-authored papers, adjacent words in a sentence, shared emails or electricity cables. This collection of edge-connected nodes is sometimes referred to as a (mathematical) **graph**. This should not be confused with the more colloquial use of the term graph to mean statistical graphic, which is why I tend to use the term 'chart' or 'visualization' to refer to the latter.

Most network visualizations will symbolise both nodes and edges, but depending on the research questions being answered, the arrangement, symbolisation and prominence given to nodes and the links between them may vary.

For examples of some of the approaches that have been taken to represent networks, have a look at the collection at [visualcomplexity.com](http://www.visualcomplexity.com/vc/) - an archive of data visualization examples that attempts _"to be a unified resource space for anyone interested in the visualization of complex networks"_. The examples range from the artistic to the functional and may provide some inspiration as how you can effectively represent network structures to answer research questions.

![Visual complexity](https://staff.city.ac.uk/~jwo/datavis2023b/session09/images/visualComplexity.jpg)

{(task|}

Browse the collection at [visualcomplexity.com](http://www.visualcomplexity.com/vc/) and identify which examples are more effective as data visualizations and which are less so. What are your criteria for judging the effectiveness of each visualization?

After doing this, have a look at Robert Kosara’s blog post [Graphs beyond the hairball](http://eagereyes.org/techniques/graphs-hairball). Does this article change your view on what is effective and what is not?

{|task)}

Perhaps more than any other category of data visualization, network visualizations are vulnerable to design for visual spectacle at the cost of any meaningful data representation. Edward Tufte’s observation, made 40 years ago, seems more relevant today than ever:

> Occasionally designers seem to seek credit merely for possessing a new technology, rather than using it to make better designs. Computers and their affiliated apparatus can do powerful things graphically, in part by turning out the hundreds of plots necessary for good data analysis. But at least a few computer graphics only evoke the response "Isn’t it remarkable that the computer can be programmed to draw like that?" instead of "My, what interesting data".

_Edward Tufte (1983), The Visual Display of Quantitative Information p.120._

The nature of many complex networks means that with suitable symbolisation intricate and beautiful graphics can be produced, but it is worth recalling from the first session in this module that in addition to producing an aesthetic engagement, good data visualization will also be a _"kind of narrative providing a clear answer to a question without extraneous details"_.

Consider the following example of two patent network visualizations created by Periscopic, one for Google and one for Apple:

![Visual complexity](https://staff.city.ac.uk/~jwo/datavis2023b/session09/images/patentsAppleGoogle.jpg)

_Patent co-inventor networks for Apple (left) and Google (right). Image &copy; Periscopic._

{(task|}

A narrative is provided by [Fast Company](https://www.fastcompany.com/3068474/the-real-difference-between-google-and-apple) who commissioned the work. How does the visualization support the narrative? What (if anything) does the visualization help you understand beyond 'Apple hierarchical, Google collaborative'?

{|task)}

How about this beautifully organic looking [visualization of a Deep Learning Network](https://www.graphcore.ai/posts/what-does-machine-learning-look-like)? What insights does the visualization provide?

![GraphCore](https://staff.city.ac.uk/~jwo/datavis2023b/session09/images/graphCore.jpg)

_Graphcore visualization of a deep learning network generated from TensorFlow._

### Case Study: London Cycle Hire network visualization

Here is an attempt to reveal network structure, in particular to answer the question, _To what extent do men and women who live in London and use the London Cycle Hire Scheme ("Santander Bikes"), make different types of journeys?_

Journeys between bicycle docking stations are represented as curved blue lines. Each curve has a straighter end (origin) and a more sharply curved end (destination). This represents the direction of flow without the need to draw arrow heads that would only clutter the picture. Journeys that start and end at the same docking station are represented as ellipses. The greater the number of journeys, the thicker and darker the line. A muted backdrop map is provided for some context (additional context is provided by mouse interaction not seen these static images).

![Cycle journeys: male](https://staff.city.ac.uk/~jwo/datavis2023b/session09/images/bikeFlowsMale.jpg)

_Cycle hire journeys made by men living within 5km of a docking station._

![Cycle journeys: female](https://staff.city.ac.uk/~jwo/datavis2023b/session09/images/bikeFlowsFemale.jpg)

_Cycle hire journeys made by women living within 5km of a docking station._

So does this help us answer our research question? We can see some structure in the flows for both men and women. For example, Hyde Park to the west of the centre of the map shows large number of journeys within the park, especially for women. There are noticeable 'hubs' of activity in the City of London and just south of the river at London Bridge, especially for men. Men seem to cross the river more frequently than women.

Further structure could be revealed by 'time-slicing' the data - that is, to see how the patterns change at different times of day or days of the week or months of the year. This requires interaction or multiple images, but the same principles of network visualization design can be applied (see [Beecham and Wood (2014)](#references) for more details).

Perhaps animation might help? Using the same principle of asymmetric curves to summarise direction, we can animate small dots along these trajectories where the brightness and density of dots represents the frequency of journeys (i.e. as a replacement for line width in the figures above):

![We are the City](https://staff.city.ac.uk/~jwo/datavis2023b/session09/images/watc.jpg)

_[We are the City: Animation](https://vimeo.com/jowood/watc)_

Does animation add anything to your understanding of the network that was not so apparent in static form? You may also wish to watch this [TEDx talk from 2012](https://www.youtube.com/watch?v=FaRBUnO5PZI) in which I showed how such visualizations of cyclists' journeys can be used to answer a variety of related research questions.

## 2. Node-link Representation

Most of the examples from visualcomplexity.com represent networks using some kind of point-based graphical symbolisation to represent nodes in the network and line-based symbolisation to represent the links between them. These rely heavily on the gestalt principle of **connectedness** that we discussed way back in Session 4 - that is, our tendency to assume that two features with a line between them are connected in some way.

{(infobox|}

Computer scientist students may have come across a form of node-link representation as part of the theory of computation, for example when representing a Finite State Machine (FSM):

```dot
digraph finite_state_machine {
  bgcolor="#ffffff00"
  node [fontname="Lexend, sans-serif"]
  edge [fontname="Lexend, sans-serif"]
  rankdir=LR;
  node [shape = doublecircle]; q1 r1;
  node [shape = circle] s q2 r2 ;
  node [shape=circle style=invis]; out;

  out -> s;
  s -> q1 [label = "a"];
  s -> r1 [label = "b"];
  q1 -> q1 [label = "a"];
  q1 -> q2 [label = "b"];
  q2 -> q2 [label = "b"];
  q2 -> q1 [label = "a"];
  r1 -> r1 [label = "b"];
  r1 -> r2 [label = "a"];
  r2 -> r2 [label = "a"];
  r2 -> r1 [label = "b"];
}
```

If you are familiar with FSMs, does the diagram help you understand the transitions between states (for example, can you determine what the machine does with string of "a"s and "b"s?) If so, why is this? If not, why not?

{|infobox)}

Vega-Lite has a somewhat limited ability to show node-link representations like this, but nevertheless it is useful to consider the steps required to assemble data to represent networks and some basic node-link visualizations. We will also see how it is possible to use [elm-vega](https://package.elm-lang.org/packages/gicentre/elm-vega/latest/) (rather than elm-vegalite) to provide more sophisticated network visualizations.

### Co-authorship networks

Suppose we wish to represent a simple co-authorship network based on the journal publications of the giCentre (the research group in the department of Computer Science focussed on visualization). The data can be found at City’s [open access publications site](https://openaccess.city.ac.uk/view/divisions/IIGIC/). The first few items in the list of publications are shown below:

---

**Andrienko, N., Andrienko, G., Chen, S. and Fisher, B.** (2022) Seeking patterns of visual pattern discovery for knowledge building. _Computer Graphics Forum_

**Bunks, C., Weyde, T., Slingsby, A. and Wood, J.** (2022) Visualization of tonal harmony for jazz lead sheets. _Computer Graphics Forum,_

**Chen, M., Jianu, R., Archambault, D., Dykes, J., Slingsby, A., Ritsos, P., Wood, J., Turkay, C. and Bach, B.** (2022) RAMPVIS: Answering the challenges of building visualisation capabilities for large-scale emergency responses. _Epidemics_

**Dykes, J., Jianu R., Slingsby, A., Bach, B., Borgo, R., Chen, M., Enright, J., Fang, H. and Wood, J.** (2022) Visualization for epidemiological modelling: Challenges, solutions,reflections and recommendations. _Philosophical Transactions of the Royal Society A: Mathematical, Physical and Engineering Sciences, 380(2235)._

**Sun, Y., Li, J., Chen, S., Andrienko, G., Andrienko, N. and Zhang, K.** (2022) A learning-based approach for efficient visualization construction. _Visual Informatics, 6(1), pp. 14-25._

**Youssef, A. and Reyes, Aldasoro C.** (2022) Computational image analysis techniques, programming languages and software platforms used in cancer research: A scoping review. _Medical Image Understanding and Analysis. MIUA 2022_.

---

If we are interested in who is collaborating with whom when writing papers, we can create a network of co-authorships by generating two datasets from the list of references. The first, representing _nodes_ is the list of all authors. The second, representing _edges_ is a list pairs of authors indicating how many times each pair of people have been involved in co-authorship of the same paper. This might be generated from the original reference list externally, or we can use some Elm and data shaping to create the tables.

### Building Nodes and Edges with Elm

The first stage is to import the publications data into a tidy `Table` from a CSV formatted file. We will separate all authors into their own columns (`Name1` to `Name9`), recognising that most papers have fewer than 9 authors, so many cells will be blank. Here are the first few rows of the table:

#### Papers table

^^^elm{m=(tableSummary 4 journalData)}^^^ (See the Appendix A for the full dataset).

We can calculate the number of times any two authors are part of the same paper (table row) with some Elm. The appendices of these notes provide the code (`authorPairs`), but in general terms we calculate all pairwise combinations of authors for each row, and add the pair to a frequency table. This allows us to create an 'edge' table:

#### Edge table

```elm {l=hidden}
edgeTable : Table
edgeTable =
    authorPairs journalData
```

^^^elm{m=(tableSummary 5 edgeTable)}^^^

To create a 'nodes' table, we need to process the `author1` and `author2` columns extracting all the names and removing duplicates (an author may be associated with more than one co-author and more than one paper). Additionally, we combine each name with the number of co-authors that person has (known as the _degree_ of the node) and the number of papers they have been involved in writing. Finally, we add a random x and y value for each person (the reason for which will become apparent below). Again, the Elm code to create the node table is provided in the appendix (`authorNames`)

#### Node table

```elm {l=hidden}
nodeTable : Table
nodeTable =
    authorNames journalData
```

^^^elm{m=(tableSummary 5 nodeTable)}^^^

We can visualize some elements of the network by showing the distribution of author (node) characteristics (excluding those authors involved in fewer than 5 papers):

```elm {v l interactive}
numPapersDist : Spec
numPapersDist =
    let
        data =
            dataFromColumns []
                << dataColumn "author" (strColumn "author" nodeTable |> strs)
                << dataColumn "numPapers" (numColumn "numPapers" nodeTable |> nums)

        trans =
            transform
                << filter (fiExpr "datum.numPapers >= 5")

        enc =
            encoding
                << position X
                    [ pName "author"
                    , pSort [ soByChannel chY, soDescending ]
                    ]
                << position Y [ pName "numPapers", pQuant, pAxis [ axGrid False ] ]
    in
    toVegaLite
        [ title "Number of papers co-written by each author" []
        , width 640
        , data []
        , trans []
        , enc []
        , bar []
        ]
```

```elm {v l interactive}
coAuthorDist : Spec
coAuthorDist =
    let
        data =
            dataFromColumns []
                << dataColumn "author" (strColumn "author" nodeTable |> strs)
                << dataColumn "numCoAuthors" (numColumn "numCoAuthors" nodeTable |> nums)
                << dataColumn "numPapers" (numColumn "numPapers" nodeTable |> nums)

        trans =
            transform
                << filter (fiExpr "datum.numPapers >= 5")

        enc =
            encoding
                << position X [ pName "author", pSort [ soByChannel chY, soDescending ] ]
                << position Y [ pName "numCoAuthors", pQuant, pAxis [ axGrid False ] ]
    in
    toVegaLite
        [ title "Total number of collaborators for each author" []
        , width 640
        , data []
        , trans []
        , enc []
        , bar []
        ]
```

For all authors (i.e no longer filtering just those involved in at least 5 papers):

```elm {v l interactive}
colabPapers : Spec
colabPapers =
    let
        data =
            dataFromColumns []
                << dataColumn "author" (strColumn "author" nodeTable |> strs)
                << dataColumn "numCoAuthors" (numColumn "numCoAuthors" nodeTable |> nums)
                << dataColumn "numPapers" (numColumn "numPapers" nodeTable |> nums)

        enc =
            encoding
                << position X [ pName "numPapers", pScale [ scType scLog ] ]
                << position Y [ pName "numCoAuthors", pScale [ scType scLog ] ]
                << tooltip [ tName "author" ]
    in
    toVegaLite [ width 640, data [], enc [], circle [] ]
```

### Visualizing the Nodes and Edges

But none of the co-author charts above shows the network as a whole. To do that we can symbolise co-authorship pairs (edges) with a line and authors (nodes) with a point symbol. Where should we place these nodes? As a first attempt we might simply place them randomly in space and then join those that have co-authored on the same paper. This is where we can use those random values `rx` and `ry` present in the node table:

```elm {v l interactive highlight=[17-22,30-36]}
nodeLink1 : Spec
nodeLink1 =
    let
        edgeData =
            dataFromColumns []
                << dataColumn "author1" (strColumn "author1" edgeTable |> strs)
                << dataColumn "author2" (strColumn "author2" edgeTable |> strs)
                << dataColumn "freq" (numColumn "freq" edgeTable |> nums)

        nodeData =
            dataFromColumns []
                << dataColumn "author" (strColumn "author" nodeTable |> strs)
                << dataColumn "numPapers" (numColumn "numPapers" nodeTable |> nums)
                << dataColumn "rx" (numColumn "rx" nodeTable |> nums)
                << dataColumn "ry" (numColumn "ry" nodeTable |> nums)

        trans =
            transform
                << lookup "author2" (nodeData []) "author" (luFields [ "rx", "ry" ])
                << calculateAs "datum.rx" "rx2"
                << calculateAs "datum.ry" "ry2"
                << lookup "author1" (nodeData []) "author" (luFields [ "rx", "ry" ])

        encEdges =
            encoding
                << position X [ pName "rx", pQuant, pAxis [] ]
                << position Y [ pName "ry", pQuant, pAxis [] ]
                << position X2 [ pName "rx2" ]
                << position Y2 [ pName "ry2" ]
                << strokeWidth
                    [ mName "freq"
                    , mQuant

                    -- Try uncommenting the line below to scale width of edges between 0.1 and 15 pixels
                    --, mScale [ scRange (raNums [ 0.1, 15 ]) ]
                    ]

        specEdges =
            asSpec [ edgeData [], trans [], encEdges [], rule [ maOpacity 0.6 ] ]

        encNodes =
            encoding
                << position X [ pName "rx", pQuant ]
                << position Y [ pName "ry", pQuant ]
                << size [ mName "numPapers", mQuant ]
                << tooltip [ tName "author" ]

        specNodes =
            asSpec [ nodeData [], encNodes [], circle [ maColor "red" ] ]

        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
    in
    toVegaLite [ cfg [], width 600, height 600, layer [ specEdges, specNodes ] ]
```

The node and edge tables are joined with the [lookup](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#lookup) function, just as we did in the previous session when joining spatial geometry files with attribute tables. Here we use the lookup to join the (random) x and y locations for each node to the list of edges that are used to draw lines between the node locations.

The width of each line is made proportional to the number of papers co-authored by any pair of people, and moving the mouse over red nodes allows individual authors to be identified. But the result is somewhat of a 'hairball' and shows us little about the nature of the co-authorship network.

We can improve things by using a much greater range of line widths so we can contrast more effectively, the difference between prolific collaborators and those with fewer co-authors from in the giCentre.

{(task|}Try removing the comment dashes from the `mScale` line above to scale line width between 0.1 and 15 pixels. What does this now tell you about the networks of collaboration in the giCentre?{|task)}

#### Other position encodings

One of the major limitations of the example above is the position of each author node is simply random and tells us little about the relationship between authors. Far better would be to use position to say something about these relationships.

This is where we hit a limitation of Vega-Lite. There are not really any options to reposition nodes other than to calculate new position data externally and import those data (replacing `rx` and `ry` above). There is software available for calculating optimal node-link layouts for network visualization. In particular, I would recommend [Gephi](https://gephi.org) if you would like to explore node-link layout possibilities.

Here, for example, is the same node-link diagram as above but this time with node positions calculated by one of the layout algorithms in Gephi.

```elm {v interactive}
nodeLink2 : Spec
nodeLink2 =
    let
        edgeData =
            dataFromColumns []
                << dataColumn "author1" (strColumn "author1" edgeTable |> strs)
                << dataColumn "author2" (strColumn "author2" edgeTable |> strs)
                << dataColumn "freq" (numColumn "freq" edgeTable |> nums)

        nodeData =
            dataFromColumns []
                << dataColumn "author" (strColumn "author" positionedNodeTable |> strs)
                << dataColumn "numPapers" (numColumn "numPapers" positionedNodeTable |> nums)
                << dataColumn "x" (numColumn "x" positionedNodeTable |> nums)
                << dataColumn "y" (numColumn "y" positionedNodeTable |> nums)

        trans =
            transform
                << lookup "author2" (nodeData []) "author" (luFields [ "x", "y" ])
                << calculateAs "datum.x" "x2"
                << calculateAs "datum.y" "y2"
                << lookup "author1" (nodeData []) "author" (luFields [ "x", "y" ])

        encEdges =
            encoding
                << position X [ pName "x", pQuant, pScale [ scNice niFalse ], pAxis [] ]
                << position Y [ pName "y", pQuant, pScale [ scNice niFalse ], pAxis [] ]
                << position X2 [ pName "x2" ]
                << position Y2 [ pName "y2" ]
                << strokeWidth
                    [ mName "freq"
                    , mQuant
                    , mScale [ scRange (raNums [ 0.1, 15 ]) ]
                    , mLegend [ leOrient loBottomRight, leOffset -2, leTitle "Number of\nco-authorships" ]
                    ]

        specEdges =
            asSpec [ edgeData [], trans [], encEdges [], rule [ maOpacity 0.6 ] ]

        encNodes =
            encoding
                << position X [ pName "x", pQuant ]
                << position Y [ pName "y", pQuant ]
                << size
                    [ mName "numPapers"
                    , mQuant
                    , mLegend [ leOrient loBottomLeft, leOffset -2, leTitle "Number\nof papers" ]
                    ]
                << tooltip [ tName "author" ]

        specNodes =
            asSpec [ nodeData [], encNodes [], circle [ maColor "red" ] ]

        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
    in
    toVegaLite [ cfg [], width 700, height 700, layer [ specEdges, specNodes ] ]
```

Vega (rather than Vega-Lite) can be used to position nodes according to a _force-directed layout_. That is, to exert an attractive or repulsive force between nodes that share a common edge. This can be used to position nodes dynamically so that nodes that share edges are closer than those without. Below is an example of using a force-directed positioning of nodes representing the characters in _Les Misérables_ with edges defining interactions between characters:

```elm {v interactive}
force1 : Spec
force1 =
    let
        ds =
            V.dataSource
                [ V.data "node-data"
                    [ V.daUrl (V.str "https://vega.github.io/vega/data/miserables.json")
                    , V.daFormat [ V.jsonProperty (V.str "nodes") ]
                    ]
                , V.data "link-data"
                    [ V.daUrl (V.str "https://vega.github.io/vega/data/miserables.json")
                    , V.daFormat [ V.jsonProperty (V.str "links") ]
                    ]
                ]

        si =
            V.signals
                << V.signal "cx" [ V.siUpdate "width /2" ]
                << V.signal "cy" [ V.siUpdate "height /2" ]
                << V.signal "nodeCharge" [ V.siValue (V.vNum -30), V.siBind (V.iRange [ V.inMin -100, V.inMax 10, V.inStep 1 ]) ]
                << V.signal "linkDistance" [ V.siValue (V.vNum 30), V.siBind (V.iRange [ V.inMin 5, V.inMax 100, V.inStep 1 ]) ]
                << V.signal "static" [ V.siValue V.vTrue, V.siBind (V.iCheckbox []) ]
                << V.signal "fix"
                    [ V.siDescription "State variable for active node fix status."
                    , V.siValue (V.vNum 0)
                    , V.siOn
                        [ V.evHandler
                            [ V.esMerge
                                [ V.esObject [ V.esMark V.symbol, V.esType V.etMouseOut, V.esFilter [ "!event.buttons" ] ]
                                , V.esObject [ V.esSource V.esWindow, V.esType V.etMouseUp ]
                                ]
                            ]
                            [ V.evUpdate "0" ]
                        , V.evHandler [ V.esObject [ V.esMark V.symbol, V.esType V.etMouseOver ] ] [ V.evUpdate "fix || 1" ]
                        , V.evHandler
                            [ V.esObject
                                [ V.esBetween [ V.esMark V.symbol, V.esType V.etMouseDown ] [ V.esSource V.esWindow, V.esType V.etMouseUp ]
                                , V.esSource V.esWindow
                                , V.esType V.etMouseMove
                                , V.esConsume V.true
                                ]
                            ]
                            [ V.evUpdate "2", V.evForce V.true ]
                        ]
                    ]
                << V.signal "node"
                    [ V.siDescription "Graph node most recently interacted with."
                    , V.siValue V.vNull
                    , V.siOn [ V.evHandler [ V.esObject [ V.esMark V.symbol, V.esType V.etMouseOver ] ] [ V.evUpdate "fix === 1 ? item() : node" ] ]
                    ]
                << V.signal "restart"
                    [ V.siDescription "Flag to restart Force simulation upon data changes."
                    , V.siValue V.vFalse
                    , V.siOn [ V.evHandler [ V.esSelector (V.strSignal "fix") ] [ V.evUpdate "fix > 1 " ] ]
                    ]

        sc =
            V.scales
                << V.scale "cScale" [ V.scType V.scOrdinal, V.scRange (V.raScheme (V.str "category20c") []) ]

        mk =
            V.marks
                << V.mark V.symbol
                    [ V.mName "nodes"
                    , V.mFrom [ V.srData (V.str "node-data") ]
                    , V.mZIndex (V.num 1)
                    , V.mOn
                        [ V.trigger "fix" [ V.tgModifyValues "node" "fix === 1 ? {fx:node.x, fy:node.y} : {fx:x(), fy:y()}" ]
                        , V.trigger "!fix" [ V.tgModifyValues "node" "{fx: null, fy: null}" ]
                        ]
                    , V.mEncode
                        [ V.enEnter
                            [ V.maFill [ V.vScale "cScale", V.vField (V.field "group") ]
                            , V.maStroke [ V.white ]
                            ]
                        , V.enUpdate
                            [ V.maSize [ V.vNum 100 ]
                            , V.maCursor [ V.cursorValue V.cuPointer ]
                            ]
                        ]
                    , V.mTransform
                        [ V.trForce
                            [ V.fsIterations (V.num 300)
                            , V.fsRestart (V.booSignal "restart")
                            , V.fsStatic (V.booSignal "static")
                            , V.fsForces
                                [ V.foCenter (V.numSignal "cx") (V.numSignal "cy")
                                , V.foCollide (V.num 10) []
                                , V.foNBody [ V.fpStrength (V.numSignal "nodeCharge") ]
                                , V.foLink (V.str "link-data") [ V.fpDistance (V.numSignal "linkDistance") ]
                                ]
                            ]
                        ]
                    ]
                << V.mark V.path
                    [ V.mFrom [ V.srData (V.str "link-data") ]
                    , V.mInteractive V.false
                    , V.mEncode
                        [ V.enUpdate
                            [ V.maStroke [ V.vStr "#ccc" ]
                            , V.maStrokeWidth [ V.vNum 0.5 ]
                            ]
                        ]
                    , V.mTransform
                        [ V.trLinkPath
                            [ V.lpShape V.lsLine
                            , V.lpSourceX (V.field "datum.source.x")
                            , V.lpSourceY (V.field "datum.source.y")
                            , V.lpTargetX (V.field "datum.target.x")
                            , V.lpTargetY (V.field "datum.target.y")
                            ]
                        ]
                    ]
    in
    V.toVega [ V.width 700, V.height 500, V.padding 0, V.autosize [ V.asNone ], ds, si [], sc [], mk [] ]
```

Alternatively we might use a simple positioning of the nodes along one dimension and symbolise edges differently. The 'arc plot' below shows the same _Les Misérables_ data, but joins connected nodes with circle arcs with a width proportional to the number of interactions:

```elm {v}
arcPlot : Spec
arcPlot =
    let
        ds =
            V.dataSource
                [ V.data "edges"
                    [ V.daUrl (V.str "https://vega.github.io/vega/data/miserables.json")
                    , V.daFormat [ V.jsonProperty (V.str "links") ]
                    ]
                , V.data "sourceDegree" [ V.daSource "edges" ]
                    |> V.transform [ V.trAggregate [ V.agGroupBy [ V.field "source" ] ] ]
                , V.data "targetDegree" [ V.daSource "edges" ]
                    |> V.transform [ V.trAggregate [ V.agGroupBy [ V.field "target" ] ] ]
                , V.data "nodes"
                    [ V.daUrl (V.str "https://vega.github.io/vega/data/miserables.json")
                    , V.daFormat [ V.jsonProperty (V.str "nodes") ]
                    ]
                    |> V.transform
                        [ V.trWindow [ V.wnOperation V.woRank "order" ] []
                        , V.trLookup "sourceDegree"
                            (V.field "source")
                            [ V.field "index" ]
                            [ V.luAs [ "sourceDegree" ], V.luDefault (V.vObject [ V.keyValue "count" (V.vNum 0) ]) ]
                        , V.trLookup "targetDegree"
                            (V.field "target")
                            [ V.field "index" ]
                            [ V.luAs [ "targetDegree" ], V.luDefault (V.vObject [ V.keyValue "count" (V.vNum 0) ]) ]
                        , V.trFormula "datum.sourceDegree.count + datum.targetDegree.count" "degree"
                        ]
                ]

        sc =
            V.scales
                << V.scale "position"
                    [ V.scType V.scBand
                    , V.scDomain (V.doData [ V.daDataset "nodes", V.daField (V.field "order"), V.daSort [] ])
                    , V.scRange V.raWidth
                    ]
                << V.scale "cScale"
                    [ V.scType V.scOrdinal
                    , V.scRange V.raCategory
                    , V.scDomain (V.doData [ V.daDataset "nodes", V.daField (V.field "group") ])
                    ]

        mk =
            V.marks
                << V.mark V.symbol
                    [ V.mName "layout"
                    , V.mInteractive V.false
                    , V.mFrom [ V.srData (V.str "nodes") ]
                    , V.mEncode
                        [ V.enEnter [ V.maOpacity [ V.vNum 0 ] ]
                        , V.enUpdate
                            [ V.maX [ V.vScale "position", V.vField (V.field "order") ]
                            , V.maY [ V.vNum 0 ]
                            , V.maSize
                                [ V.vField (V.field "degree")
                                , V.vMultiply (V.vNum 5)
                                , V.vOffset (V.vNum 10)
                                ]
                            , V.maFill [ V.vScale "cScale", V.vField (V.field "group") ]
                            ]
                        ]
                    ]
                << V.mark V.path
                    [ V.mFrom [ V.srData (V.str "edges") ]
                    , V.mEncode
                        [ V.enUpdate
                            [ V.maStroke [ V.black ]
                            , V.maStrokeOpacity [ V.vNum 0.2 ]
                            , V.maStrokeWidth [ V.vField (V.field "value") ]
                            ]
                        ]
                    , V.mTransform
                        [ V.trLookup "layout"
                            (V.field "datum.index")
                            [ V.field "datum.source", V.field "datum.target" ]
                            [ V.luAs [ "sourceNode", "targetNode" ] ]
                        , V.trLinkPath
                            [ V.lpSourceX (V.fExpr "min(datum.sourceNode.x, datum.targetNode.x)")
                            , V.lpTargetX (V.fExpr "max(datum.sourceNode.x, datum.targetNode.x)")
                            , V.lpSourceY (V.fExpr "0")
                            , V.lpTargetY (V.fExpr "0")
                            , V.lpShape V.lsArc
                            ]
                        ]
                    ]
                << V.mark V.symbol
                    [ V.mFrom [ V.srData (V.str "layout") ]
                    , V.mEncode
                        [ V.enUpdate
                            [ V.maX [ V.vField (V.field "x") ]
                            , V.maY [ V.vField (V.field "y") ]
                            , V.maFill [ V.vField (V.field "fill") ]
                            , V.maSize [ V.vField (V.field "size") ]
                            ]
                        ]
                    ]
                << V.mark V.text
                    [ V.mFrom [ V.srData (V.str "nodes") ]
                    , V.mEncode
                        [ V.enUpdate
                            [ V.maX [ V.vScale "position", V.vField (V.field "order") ]
                            , V.maY [ V.vNum 7 ]
                            , V.maText [ V.vField (V.field "name") ]
                            , V.maFontSize [ V.vNum 9 ]
                            , V.maAngle [ V.vNum -90 ]
                            , V.maAlign [ V.hRight ]
                            , V.maBaseline [ V.vMiddle ]
                            ]
                        ]
                    ]
    in
    V.toVega [ V.width 770, V.padding 5, ds, sc [], mk [] ]
```

{(task|}Does the arc layout offer any advantages over a 2d node-link layout? How scalable is the approach?{|task)}

Showing the nodes and edges as a set of connected point symbols and lines makes intuitive sense – the characteristics of the visual form is strongly related to the characteristics of the network. And that similarity means that we can easily understand visualizations of simple networks using this approach.

However, it soon becomes obvious that for larger more complex networks, even with modifications such as curved and transparent lines, the visualization soon just looks too much like a 'hairball' to make sense of it. In other words, this form of visualization is not usefully scalable. Additionally, node-link visualizations can produce many different layouts of the same underlying network, so it can be difficult to know whether patterns observed are a property of the data (which is what we want) or the visualization (which we don’t). For these and other reasons, many alternatives have been proposed, such as the [Hive Plot](http://www.hiveplot.net) (Krzywinski et al, 2012).

![Hive plot](https://staff.city.ac.uk/~jwo/datavis2023b/session09/images/hivePlot.png)

_Hive plot showing gene expression network (from Krzywinski et al, 2012). Edges shown as curved lines; nodes positioned along three axes according to properties that reflect the research question._

## 3. Sankey Diagrams

Often we wish to judge not just how nodes in a network are connected, but make informed judgements about the relative magnitudes of connections. As we have already seen, we can use line width to indicate magnitude of a connection (number of co-publications, amount of migration etc.).

Additionally we may wish to show how a node is connected to several others and how the magnitude of the flow from that node is split between them. A chart that uses line width and line bifurcation to indicate this is often referred to as a [Sankey diagram](https://en.wikipedia.org/wiki/Sankey_diagram).

![Access to Higher Education, Sankey diagram](https://staff.city.ac.uk/~jwo/datavis2023b/session09/images/sankeyHE.jpg)
_Sankey Diagram showing access to Higher Education by deprivation quintile and subject.
[Source and further details](https://public.tableau.com/profile/westlake.cjw#!/vizhome/HE_Viz/AccesstoHE)_

### Implementation (advanced)

Full Sankey diagrams are challenging to produce in Vega-Lite (although an approximation can be built using the [trail mark](https://vega.github.io/vega-lite/docs/trail.html)). We can though use [elm-vega](https://package.elm-lang.org/packages/gicentre/elm-vega/latest/) (as opposed to elm-vegalite) to generate interactive two-state Sankey diagrams.

Appendix E contains an Elm function `sankeyBuilder` that uses elm-vega to generate interactive Sankey diagrams (adapted from [this tutorial example](https://www.elastic.co/blog/sankey-visualization-with-vega-in-kibana) by Yuri Astrakhan). It can be used by providing a dataset that should contain paired transitions between an 'origin' field (identified with `oName`), 'destination' (identified with `dName`) and a magnitude/value associated with that transition (identified with `vName`).

Suppose we had information on the number of people in different salary bands in both 2012 and 2022.

```elm {l=hidden}
salaryTable : Table
salaryTable =
    """salary2012,salary2022,numPeople
20-30k,leaver,4
20-30k,20-30k,15
20-30k,30-40k,5
20-30k,40-50k,3
30-40k,30-40k,10
30-40k,40-50k,5
40-50k,40-50k,3"""
        |> fromCSV
```

^^^elm{m=(tableSummary -1 salaryTable)}^^^

The origin and destination columns in the Sankey diagram are labelled with `oLabel` and `dLabel` (this also uses an Elm pattern you won't have seen yet where a default [record](https://elm-lang.org/docs/records) encased in curly braces is modified with non default options after a vertical bar `|`).

```elm {l v interactive}
sankeySalary : Spec
sankeySalary =
    let
        myData =
            V.dataFromColumns "sankeyData" []
                << V.dataColumn "salary2012" (V.vStrs (strColumn "salary2012" salaryTable))
                << V.dataColumn "salary2022" (V.vStrs (strColumn "salary2022" salaryTable))
                << V.dataColumn "numPeople" (V.vNums (numColumn "numPeople" salaryTable))
    in
    sankeyBuilder
        { defaultSankey
            | data = myData []
            , oName = "salary2012"
            , dName = "salary2022"
            , vName = "numPeople"
            , height = 200
            , oLabel = "2012"
            , dLabel = "2022"
        }
```

Clicking on either the origin (left) or destination (right) stacks filters the diagram by stacked group. Double click a stack to revert to the full Sankey diagram. Hovering over a group in either the origin or destination stacks highlights the flows from or to the stack group respectively.

Here's another example using polling on political party voting preference that allows us to see recent party preferences broken down by which party respondents voted for in the 2019 General Election (data source: [YouGov 2022](https://docs.cdn.yougov.com/9au6aflu7r/TheTimes_VI_221102_W.pdf)). It uses the `cScale` option to provide custom categorical colour scheme that uses conventional UK party colours.

```elm {l=hidden}
votingTable : Table
votingTable =
    """ge2019,today,respondents
Labour,Labour,344
Labour,Conservative,4
Labour,Liberal Democrats,9
Labour,Green,9
Labour,Reform UK,4
Labour,will not vote,13
Labour,don't know,35
Conservative,Labour,70
Conservative,Conservative,233
Conservative,Liberal Democrats,23
Conservative,Green,12
Conservative,Reform UK,47
Conservative,will not vote,41
Conservative,don't know,140
Liberal Democrats,Labour,51
Liberal Democrats,Conservative,10
Liberal Democrats,Liberal Democrats,59
Liberal Democrats,Green,11
Liberal Democrats,Reform UK,2
Liberal Democrats,will not vote,2
Liberal Democrats,don't know,22
"""
        |> fromCSV
```

^^^elm{m=(tableSummary -1 votingTable)}^^^

```elm {l v interactive}
votingChanges : Spec
votingChanges =
    let
        votingData =
            V.dataFromColumns "sankeyData" []
                << V.dataColumn "ge2019" (V.vStrs (strColumn "ge2019" votingTable))
                << V.dataColumn "today" (V.vStrs (strColumn "today" votingTable))
                << V.dataColumn "respondents" (V.vNums (numColumn "respondents" votingTable))
    in
    sankeyBuilder
        { defaultSankey
            | data = votingData []
            , oName = "ge2019"
            , dName = "today"
            , vName = "respondents"
            , height = 500
            , oLabel = "General Election 2019"
            , dLabel = "Current voting intention"
            , cScale =
                categoricalDomainMap
                    [ ( "Labour", "rgb(209,45,65)" )
                    , ( "Conservative", "rgb(58,134,214)" )
                    , ( "Liberal Democrats", "rgb(242,196,73)" )
                    , ( "Green", "rgb(50,200,50)" )
                    , ( "Reform UK", "rgb(85,184,207)" )
                    , ( "will not vote", "rgb(100,100,100)" )
                    , ( "don't know", "rgb(180,180,180)" )
                    ]
        }
```

And as a final example, the giCentre co-authorship we visualized earlier (edgeTable).

```elm {l v interactive}
sankeyCoAuthor : Spec
sankeyCoAuthor =
    let
        coAuthorData =
            V.dataFromColumns "sankeyData" []
                << V.dataColumn "author1" (V.vStrs (strColumn "author1" edgeTable))
                << V.dataColumn "author2" (V.vStrs (strColumn "author2" edgeTable))
                << V.dataColumn "freq" (V.vNums (numColumn "freq" edgeTable))
    in
    sankeyBuilder
        { defaultSankey
            | data = coAuthorData []
            , oName = "author1"
            , dName = "author2"
            , vName = "freq"
            , height = 1000
            , oLabel = "Author 1"
            , dLabel = "Author 2"
        }
```

## 4. Matrix View of Networks

As Robert Kosara noted (see the question/task at the start of these notes), node-link diagrams such as the ones above do not scale well to larger more complex networks. They soon become tangled 'hairballs' where identifying the structure of the connections between items is difficult. Force-directed layouts, curved edges and clustering can all help with this problem but they do not eliminate it entirely.

An alternative approach we will consider here comes from the observation that any graph (in the mathematical sense) can also be represented as a matrix. See for example [Viewing Matrices and Probability as Graphs](https://www.math3ma.com/blog/matrices-probability-graphs). In the visualization context, we can represent edges not by lines, but by their position in a matrix. Such matrix views are much more scalable in that we use colour or point symbol rather than number of lines to represent the volume of connections between items. Each node is represented as a row and a column in the grid. Each cell in the matrix is therefore the intersection of two node positions. That cell can then be symbolised according to the number of connections between them.

To return to the giCentre co-authorship network, we represent all authors along the rows and columns of a matrix and the cell of intersecting row and column values represents the number of papers co-authored by the pair of authors.

```elm {v interactive}
matrixLayout : Spec
matrixLayout =
    let
        edgeData =
            dataFromColumns []
                << dataColumn "author1" (strColumn "author1" edgeTable |> strs)
                << dataColumn "author2" (strColumn "author2" edgeTable |> strs)
                << dataColumn "freq" (numColumn "freq" edgeTable |> nums)

        nodeData =
            dataFromColumns []
                << dataColumn "author" (strColumn "author" nodeTable |> strs)
                << dataColumn "order"
                    (List.range 1 (List.length (strColumn "author" nodeTable))
                        |> List.map toFloat
                        |> nums
                    )
                << dataColumn "numPapers" (numColumn "numPapers" nodeTable |> nums)

        trans =
            transform
                << lookup "author1" (nodeData []) "author" (luFields [ "order" ])
                << calculateAs "datum.order" "colOrder"
                << lookup "author2" (nodeData []) "author" (luFields [ "order" ])
                << calculateAs "datum.order" "rowOrder"

        encMatrixBL =
            encoding
                << position X [ pName "colOrder", pAxis [] ]
                << position Y [ pName "rowOrder", pAxis [] ]
                << size [ mName "freq", mQuant ]
                << tooltips [ [ tName "author1" ], [ tName "author2" ] ]

        specMatrixBL =
            asSpec [ edgeData [], trans [], encMatrixBL [], circle [] ]

        encMatrixTR =
            encoding
                << position X [ pName "colOrder", pAxis [] ]
                << position Y [ pName "rowOrder", pAxis [] ]
                << size [ mName "freq", mQuant ]
                << tooltips [ [ tName "author1" ], [ tName "author2" ] ]

        specMatrixTR =
            asSpec [ edgeData [], trans [], encMatrixTR [], circle [] ]

        encRowLabels =
            encoding
                << position X [ pNum 0 ]
                << position Y [ pName "order", pAxis [] ]
                << text [ tName "author" ]

        specRowLabels =
            asSpec [ nodeData [], encRowLabels [], textMark [ maAlign haRight, maSize 3, maDx -5 ] ]

        encColLabels =
            encoding
                << position X [ pName "order", pAxis [] ]
                << position Y [ pNum 0 ]
                << text [ tName "author" ]

        specColLabels =
            asSpec [ nodeData [], encColLabels [], textMark [ maAlign haRight, maAngle 90, maSize 3 ] ]
    in
    toVegaLite [ width 1200, height 1200, layer [ specMatrixTR, specRowLabels, specColLabels ] ]
```

While matrices can be helpful in eliminating the large numbers of lines that would otherwise be shown in a node-link representation, they don't always take advantage of position to encode useful characteristics of the data. This is particularly the case when nodes represent features with some geospatial positioning.

For example, consider the bicycle flows shown at the start of this session. If we were to represent each bicycle docking station as a row and as a column in a matrix, each cell could be used to record the number of journeys between any two docking stations. The challenge becomes, how best to order the rows and columns to capture the spatial arrangement of the docking stations.

One approach is to use the [OD map](#references), that re-orders matrix cells hierarchically to preserve the spatial arrangement of grid cell locations. To see how to construct an OD map in litvis/Vega-Lite, let's consider a simple case of movement between London boroughs...

### Conventional Flow Mapping

Suppose we have some data that represent flows of some kind between a set of origin (O) and destination (D) locations. For example, we might have a record of some journeys in London:

```elm {l=hidden}
simpleFlow : Table
simpleFlow =
    """from,to,flow
City,Hackney,2
Hackney,City,1
Hackney,Hackney,1
Hackney,Islington,1
Hackney,Newham,1
Islington,Waltham Forest,3
Newham,City,1
Newham,Redbridge,1
Redbridge,Hackney,1
Waltham Forest,Islington,1
Waltham Forest,Newham,1
Waltham Forest,Redbridge,1
Waltham Forest,Waltham Forest,1"""
        |> fromCSV


simpleFlowData : List DataColumn -> Data
simpleFlowData =
    dataFromColumns []
        << dataColumn "from" (strs (strColumn "from" simpleFlow))
        << dataColumn "to" (strs (strColumn "to" simpleFlow))
        << dataColumn "flow" (nums (numColumn "flow" simpleFlow))
```

^^^elm {m=(tableSummary 3 simpleFlow)}^^^

We are likely to have another table containing location information that may be grid cell based, geographic or both:

```elm {l=hidden}
simpleGrid : Table
simpleGrid =
    """borough,gridCol,gridRow,longitude,latitude
    City,1,2,-0.0911,51.5145
    Hackney,1,1,-0.0633,51.5526
    Islington,0,1,-0.1102,51.5482
    Newham,2,1,0.0368,51.5282
    Redbridge,2,0,0.0758,51.5851
    Waltham Forest,1,0,-0.0123,51.5937"""
        |> fromCSV


simpleGridData : List DataColumn -> Data
simpleGridData =
    dataFromColumns []
        << dataColumn "borough" (strs (strColumn "borough" simpleGrid))
        << dataColumn "gridCol" (nums (numColumn "gridCol" simpleGrid))
        << dataColumn "gridRow" (nums (numColumn "gridRow" simpleGrid))
        << dataColumn "longitude" (nums (numColumn "longitude" simpleGrid))
        << dataColumn "latitude" (nums (numColumn "latitude" simpleGrid))
```

^^^elm {m=(tableSummary -1 simpleGrid)}^^^

To construct any form of flow mapping we need to join the two tables. Here is the join 'at source' using the [tidy package](https://package.elm-lang.org/packages/gicentre/tidy/latest/Tidy). In other words we create the joined data table before passing it to the visualization specification:

```elm {l}
joinODLocationTable : String -> String -> Table
joinODLocationTable xField yField =
    let
        t1 =
            leftJoin ( simpleFlow, "from" ) ( simpleGrid, "borough" )
                |> renameColumn xField "oX"
                |> renameColumn yField "oY"
    in
    leftJoin ( t1, "to" ) ( simpleGrid, "borough" )
        |> renameColumn xField "dX"
        |> renameColumn yField "dY"
        |> filterColumns (\s -> List.member s [ "from", "flow", "to", "oX", "oY", "dX", "dY" ])
```

^^^elm {m=(tableSummary 3 (joinODLocationTable "longitude" "latitude"))}^^^

Alternatively, if you were using data from a remote source, you can do the joining within the vis specification using the two separate tables as input:

```elm {l}
joinODLocation xField yField =
    transform
        << lookup "from" (simpleGridData []) "borough" (luFields [ xField, yField ])
        << calculateAs ("datum." ++ xField) "oX"
        << calculateAs ("datum." ++ yField) "oY"
        << lookup "to" (simpleGridData []) "borough" (luFields [ xField, yField ])
        << calculateAs ("datum." ++ xField) "dX"
        << calculateAs ("datum." ++ yField) "dY"
```

Now we have pairs of origin and destination coordinates, from which we can create a simple flow map, joining origins and destinations with a [rule](https://vega.github.io/vega-lite/docs/rule.html) (straight line) mark. This has the advantage that the mark can take an `X`,`Y` or `Longitude`,`Latitude` pair representing one end of the line and another `X2`,`Y2` or `Longitude2`, `Latitude2` pair for the other. We can show flow magnitude by encoding it as the rule's stroke width:

```elm {v}
simpleFlowMapRule : Spec
simpleFlowMapRule =
    let
        cfg =
            configure << configuration (coView [ vicoStroke Nothing ])

        mapSpec =
            asSpec
                [ dataFromUrl "https://gicentre.github.io/data/geoTutorials/londonBoroughsSelected.json"
                    [ topojsonFeature "boroughs" ]
                , geoshape [ maStroke "white", maStrokeWidth 2, maOpacity 0.2 ]
                ]

        flowEnc =
            encoding
                << position Longitude [ pName "oX", pQuant ]
                << position Latitude [ pName "oY", pQuant ]
                << position Longitude2 [ pName "dX" ]
                << position Latitude2 [ pName "dY" ]
                << strokeWidth
                    [ mName "flow"
                    , mQuant
                    , mLegend [ leTickCount 2, leOrient loTopLeft ]
                    , mScale [ scRange (raNums [ 1, 10 ]) ]
                    ]

        flowSpec =
            asSpec
                [ simpleFlowData []
                , joinODLocation "longitude" "latitude" []
                , flowEnc []
                , rule [ maColor "brown", maOpacity 0.5, maStrokeCap caRound ]
                ]

        labelEnc =
            encoding
                << position Longitude [ pName "longitude", pQuant ]
                << position Latitude [ pName "latitude", pQuant ]
                << text [ tName "borough", tNominal ]

        mapLabelSpec =
            asSpec [ simpleGridData [], labelEnc [], textMark [ maDy 10 ] ]
    in
    toVegaLite
        [ cfg []
        , width 500
        , height 400
        , layer [ mapSpec, mapLabelSpec, flowSpec ]
        ]
```

We could show the same flows on a grid (see 'Relaxing Geography' from last week), again using the rule mark to join origins and destinations:

```elm {l=hidden}
labelledGridSpec : Spec
labelledGridSpec =
    let
        posEnc =
            encoding
                << position X [ pName "gridCol", pQuant, pAxis [] ]
                << position Y [ pName "gridRow", pQuant, pSort [ soDescending ], pAxis [] ]

        gridSpec =
            asSpec [ square [ maStroke "white", maStrokeWidth 3, maFill "#dde4ed", maSize 10000 ] ]

        labelEnc =
            encoding
                << text [ tName "borough", tNominal ]

        labelSpec =
            asSpec [ labelEnc [], textMark [ maFontSize 10, maDy 15 ] ]
    in
    asSpec [ simpleGridData [], posEnc [], layer [ gridSpec, labelSpec ] ]
```

```elm {v}
simpleFlowGridRule : Spec
simpleFlowGridRule =
    let
        cfg =
            configure << configuration (coView [ vicoStroke Nothing ])

        -- Flow lines
        flowEnc =
            encoding
                << position X [ pName "oX", pQuant, pAxis [] ]
                << position Y [ pName "oY", pQuant, pSort [ soDescending ], pAxis [] ]
                << position X2 [ pName "dX" ]
                << position Y2 [ pName "dY" ]
                << strokeWidth [ mName "flow", mQuant, mLegend [], mScale [ scRange (raNums [ 1, 10 ]) ] ]

        flowSpec =
            asSpec
                [ simpleFlowData []
                , joinODLocation "gridCol" "gridRow" []
                , flowEnc []
                , rule [ maColor "brown", maOpacity 0.5, maStrokeCap caRound ]
                ]
    in
    toVegaLite
        [ cfg []
        , background "#f9f9fc"
        , layer [ labelledGridSpec, flowSpec ]
        ]
```

We have a few problems with these representations

- Direction of flow is not symbolised.
- Co-linearity makes it hard to distinguish some flows (e.g. Islington - Hackney - Newham in the grid map).
- Flows that start and end at the same location are not shown.
- Flows over longer distances are more visually dominant than shorter ones ('distance saliency').

#### Arrow flows

We can adapt the previous flow maps (geo or grid) to use the [trail](https://vega.github.io/vega-lite/docs/trail.html) mark instead of [rule](https://vega.github.io/vega-lite/docs/rule.html) mark. A trail allows us to change the width of a line symbol along its length by setting a different size at each end of the OD line.

To do this we need to _tidy_ our OD table so that the x and y coordinates of all nodes stored as a single pair of fields. We can add additional fields to identify whether a coordinate pair is an origin or destination and some identifier to link each O and D location.

```elm {l}
tidyODTable : Table -> Table
tidyODTable =
    insertIndexColumn "id" ""
        >> gather "odType" "value" [ ( "oX", "ox" ), ( "oY", "oy" ), ( "dX", "dx" ), ( "dY", "dy" ) ]
        >> bisect "odType" headTail ( "od", "type" )
        >> spread "type" "value"
```

^^^elm {m=(tableSummary 6 (tidyODTable (joinODLocationTable "longitude" "latitude")))}^^^

For data not stored in local tables, we could instead apply an equivalent tidying process within the vega-lite specification:

```elm {l}
odTrans xField yField =
    joinODLocation xField yField
        << foldAs [ "oX", "dX" ] "colId" "col"
        << foldAs [ "oY", "dY" ] "rowId" "row"
        << filter (fiExpr "substring(datum.colId,0,1) == substring(datum.rowId,0,1)")
        << window [ ( [ wiOp woRowNumber ], "tableRow" ) ] []
        << calculateAs "round(datum.tableRow / 2)" "id"
```

```elm {v}
simpleFlowGridTrail : Spec
simpleFlowGridTrail =
    let
        cfg =
            configure << configuration (coView [ vicoStroke Nothing ])

        table =
            joinODLocationTable "gridCol" "gridRow" |> tidyODTable

        odData =
            dataFromColumns []
                << dataColumn "id" (strColumn "id" table |> strs)
                << dataColumn "od" (strColumn "od" table |> strs)
                << dataColumn "x" (numColumn "x" table |> nums)
                << dataColumn "y" (numColumn "y" table |> nums)
                << dataColumn "flow" (numColumn "flow" table |> nums)

        trans =
            joinODLocation "gridCol" "gridRow"
                << calculateAs "datum.od == 'o' ? datum.flow*2 : datum.flow/2" "w"

        -- Flow lines
        flowEnc =
            encoding
                << position X [ pName "x", pQuant ]
                << position Y [ pName "y", pQuant ]
                << detail [ dName "id" ]
                << size [ mName "w", mQuant, mScale [ scRange (raNums [ 0, 15 ]) ], mLegend [] ]

        flowSpec =
            asSpec
                [ odData []
                , trans []
                , flowEnc []
                , trail [ maColor "brown", maOpacity 0.5, maStrokeCap caRound ]
                ]
    in
    toVegaLite
        [ cfg []
        , background "#f9f9fc"
        , layer [ labelledGridSpec, flowSpec ]
        ]
```

The trails allow us to represent each flow direction as an arrow moving from the 'fat' to 'thin' end of each symbol. This in theory allows us to separate the directions between any pair of points, but it remains a little hard to decode with co-linear symbols.

#### Asymmetric Curves

Following the approach of [Fekete et al, 2003](http://www.cs.umd.edu/hcil/trs/2003-32/2003-32.pdf), we can make the flow lines curved by adding extra vertices that fall outside of the linear path from origin to destination. The 'long' end of each curve represents the origin, the 'short' more sharply curving end the destination.

We can create an Elm function that given origin and destination locations, returns an additional point 25% along the line from the destination, offset by 30 degrees in direction of travel:

```elm {l}
ctrlPoint : ( Float, Float ) -> ( Float, Float ) -> ( String, String )
ctrlPoint ( x1, y1 ) ( x2, y2 ) =
    let
        curveAngle =
            degrees -30

        x =
            (x1 - x2) / 4

        y =
            (y1 - y2) / 4
    in
    ( x2 + x * cos curveAngle - y * sin curveAngle |> String.fromFloat
    , y2 + y * cos curveAngle + x * sin curveAngle |> String.fromFloat
    )
```

We can then insert that extra location into the table of OD locations:

```elm {l}
ctrlPointTable : String -> String -> Table
ctrlPointTable xField yField =
    let
        tbl =
            joinODLocationTable xField yField

        oCoords =
            List.map2 Tuple.pair (numColumn "oX" tbl) (numColumn "oY" tbl)

        dCoords =
            List.map2 Tuple.pair (numColumn "dX" tbl) (numColumn "dY" tbl)

        ( ex, ey ) =
            List.map2 ctrlPoint oCoords dCoords |> List.unzip
    in
    tbl
        |> insertColumn "eX" ex
        |> insertColumn "eY" ey
        |> tidyODTable2
```

Tidying the new table now additionally involves gathering the new extra point `(ex,ey)` along with the origin and destination:

```elm {l}
tidyODTable2 : Table -> Table
tidyODTable2 =
    insertIndexColumn "id" ""
        >> gather "odType"
            "value"
            [ ( "oX", "ox" )
            , ( "oY", "oy" )
            , ( "eX", "ex" )
            , ( "eY", "ey" )
            , ( "dX", "dx" )
            , ( "dY", "dy" )
            ]
        >> bisect "odType" headTail ( "od", "type" )
        >> spread "type" "value"
```

To display the curved lines we use a [b-spline (basis)](https://vega.github.io/vega-lite/docs/line.html#properties) interpolator. Note that the order of plotting of the three coordinates per line is ensured by giving each coordinate a label in alphabetical sequence (`d`, `e`, `o`).

```elm {v}
simpleFlowGridCurve : Spec
simpleFlowGridCurve =
    let
        cfg =
            configure << configuration (coView [ vicoStroke Nothing ])

        table =
            ctrlPointTable "gridCol" "gridRow"

        odData =
            dataFromColumns []
                << dataColumn "id" (strColumn "id" table |> strs)
                << dataColumn "od" (strColumn "od" table |> strs)
                << dataColumn "x" (numColumn "x" table |> nums)
                << dataColumn "y" (numColumn "y" table |> nums)
                << dataColumn "flow" (numColumn "flow" table |> nums)

        flowEnc =
            encoding
                << position X
                    [ pName "x"
                    , pQuant
                    , pScale [ scNice niFalse, scDomain (doNums [ 0, 2 ]) ]
                    ]
                << position Y
                    [ pName "y"
                    , pQuant
                    , pScale [ scNice niFalse, scDomain (doNums [ 0, 2 ]) ]
                    ]
                << order [ oName "od", oOrdinal ]
                << detail [ dName "id" ]
                << strokeWidth [ mName "flow", mQuant, mScale [ scRange (raNums [ 1, 10 ]) ], mLegend [] ]

        flowSpec =
            asSpec
                [ odData []
                , flowEnc []
                , line
                    [ maColor "brown"
                    , maOpacity 0.5
                    , maStrokeCap caRound
                    , maInterpolate miBasis
                    ]
                ]
    in
    toVegaLite
        [ cfg []
        , background "#f9f9fc"
        , layer [ labelledGridSpec, flowSpec ]
        ]
```

These asymmetric curves have the advantage that they clearly separate flow directions (see the bi-directional movements between Islington – Waltham Forest and Hackney – City). There is a minor cost in learning to read the direction of the curve. Is this cost outweighed by its benefits?

#### OD Mapping: Placing grid maps inside grid maps

Despite possible improvements in symbolisation, we still have a problem of _distance saliency_ and, for more complex flows, the risk of 'hairball' maps with many overlapping lines. And we still cannot see any flows that start and end at the same location.

Suppose we wished to look at all the journeys that ended up in Hackney. Instead of drawing lines, we could colour each grid cell according to the number of times a journey starts in the place indicated by the cell and finishes in Hackney:

```elm {l=hidden}
odHackneyTable : Table
odHackneyTable =
    joinODLocationTable "gridCol" "gridRow"
        |> filterRows "to" ((==) "Hackney")
```

^^^elm m=(tableSummary -1 odHackneyTable)^^^

```elm {v}
gridMap : Spec
gridMap =
    let
        flowData =
            dataFromColumns []
                << dataColumn "oCol" (nums (numColumn "oX" odHackneyTable))
                << dataColumn "oRow" (nums (numColumn "oY" odHackneyTable))
                << dataColumn "dCol" (nums (numColumn "dX" odHackneyTable))
                << dataColumn "dRow" (nums (numColumn "dY" odHackneyTable))
                << dataColumn "flow" (nums (numColumn "flow" odHackneyTable))

        trans =
            transform
                -- Only show  flows that end in Hackney (at grid [1,1])
                << filter (fiExpr "datum.dCol==1 && datum.dRow==1")

        labelTrans =
            transform
                << calculateAs "slice(datum.borough,0,1)" "initial"

        enc =
            encoding
                << position X [ pName "oCol", pAxis [] ]
                << position Y [ pName "oRow", pAxis [] ]
                << color [ mName "flow", mOrdinal, mTitle "Number of journeys" ]

        odSpec =
            asSpec
                [ trans []
                , enc []
                , square [ maSize (200 * 200 / 9), maOpacity 1, maStroke "#fff" ]
                ]

        labelEnc =
            encoding
                << position X [ pName "gridCol" ]
                << position Y [ pName "gridRow" ]
                << text [ tName "initial" ]

        labelSpec =
            asSpec [ simpleGridData [], labelTrans [], labelEnc [], textMark [] ]
    in
    toVegaLite [ width 200, height 200, flowData [], layer [ odSpec, labelSpec ] ]
```

For the first time, we can now see the Hackney-Hackney journey (lighter blue cell in centre). We can see that there are two flows from City to Hackney (darker blue cell) and one from Redbridge to Hackney. The remaining white cells represent either locations with no journeys to Hackney (W, I and N) or gaps in the 3x3 grid that do not represent a location (completely blank).

But what about all the journeys that don't end in Hackney? We can repeat the 3x3 grid for each of the other destination locations, arranging those grids themselves using the same 3x3 geographic grid layout. This is achievable by [faceting](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#facet) the grid by the destination location:

```elm {l=hidden}
odGridTable : Table
odGridTable =
    joinODLocationTable "gridCol" "gridRow"
```

```elm {l=hidden}
odMap : String -> String -> Spec
odMap large small =
    let
        cfg =
            configure
                << configuration (coHeader [ hdLabelFontSize 0, hdTitle "" ])
                << configuration (coFacet [ facoSpacing 5 ])
                << configuration (coAxis [ axcoTitleX 3 ])

        flowData =
            dataFromColumns []
                << dataColumn "oCol" (nums (numColumn "oX" odGridTable))
                << dataColumn "oRow" (nums (numColumn "oY" odGridTable))
                << dataColumn "dCol" (nums (numColumn "dX" odGridTable))
                << dataColumn "dRow" (nums (numColumn "dY" odGridTable))
                << dataColumn "flow" (nums (numColumn "flow" odGridTable))
                << dataColumn "o" (strs (strColumn "from" odGridTable))
                << dataColumn "d" (strs (strColumn "to" odGridTable))

        trans =
            transform
                << calculateAs ("slice(datum." ++ large ++ ",0,1)") "initial"

        enc =
            encoding
                << position X [ pName (small ++ "Col"), pAxis [] ]
                << position Y [ pName (small ++ "Row"), pAxis [] ]
                << color [ mName "flow", mOrdinal, mTitle "Number of journeys" ]

        cellSpec =
            asSpec
                [ enc []
                , square [ maSize (200 * 200 / 110), maOpacity 1, maStrokeWidth 1.5 ]
                ]

        homeEnc =
            encoding
                << position Y [ pName (large ++ "Row"), pOrdinal, pAxis [] ]
                << position X [ pName (large ++ "Col"), pOrdinal, pAxis [] ]
                << text [ tName "initial" ]

        homeSpec =
            asSpec [ homeEnc [], textMark [ maOpacity 0.3 ] ]

        odSpec =
            asSpec [ layer [ cellSpec, homeSpec ] ]
    in
    toVegaLite
        [ cfg []
        , title ("Large cells: " ++ large ++ ", small cells: " ++ small) []
        , flowData []
        , trans []
        , facet
            [ rowBy [ fName (large ++ "Row"), fOrdinal ]
            , columnBy [ fName (large ++ "Col"), fOrdinal ]
            ]
        , specification odSpec
        ]
```

^^^elm {v=(odMap "d" "o")}^^^

Borough labels are positioned where the origin and destination labels match. For example, Redbridge (R) occupies the top-right cell so is located in the large origin top-right square and within that, labelled in the small destination top-right square. These labels provide the 'home' cells and where coloured, show flows within the same borough. The cells that are closest to these home cells are their nearest neighbours.

Only Waltham Forest (W) and Hackney (H) have within-borough flows indicated (colours under the labelled home cells), and there are no flows in the entire map longer than 1 cell apart (e.g. no flows between W and C and I to N).

We can easily transform the grid so that large cells instead represent origins and small cells represent the destinations of each flow:

^^^elm {v=(odMap "o" "d")}^^^

It is useful to note that swapping origins for destinations in this way shows exactly the same flows (same number of coloured cells, each with the same colours) but their arrangement has changed. Comparing the two sets of grid arrangements allows us to see asymmetries on the flows. For example, City (bottom centre) has incoming flows from two boroughs (Hackney and Newham) but outgoing flows only to one borough (Hackney). This was not detectable in the original node-link representation.

While the OD map can show flows not detectable with a conventional node-link network diagram, it is perhaps not as intuitive to understand, requiring time to learn how to interpret. Using colour rather than line width to show flow magnitude is less discriminating. However, its real advantage comes when showing larger datasets with many more flows...

#### OD Mapping: London Commuter Flows

Working with a more realistic dataset, we can map [London commuting patterns from the 2001 census](https://data.london.gov.uk/dataset/33e8b8a3-ee75-4819-9f0a-feb88c0cff45).

The data are provided as an _OD Matrix_ (as we had for the co-authorships earlier), which contain all the necessary flow values, but not arranged spatially. Data rows are origin boroughs and data columns destination boroughs. Other than the first labelling row and first labelling column, values represent the numbers of people who commuted between pairs of London boroughs according to the 2001 census.

^^^elm{m=(tableSummary 6 odMatrix)}^^^

The data are not yet in [tidy](https://package.elm-lang.org/packages/gicentre/tidy/latest/) format. We need to place all the commuter counts into a single column, along with columns representing their origin and destination codes by [gathering columns](https://package.elm-lang.org/packages/gicentre/tidy/latest/Tidy#gather).

```elm {l=hidden}
tidyOd : Table
tidyOd =
    odMatrix
        |> renameColumn "Origin Code" "o"
        |> gather "d"
            "numJourneys"
            (strColumn "Origin Code" odMatrix |> List.map (\s -> ( s, s )))
```

^^^elm{m=(tableSummary 6 tidyOd)}^^^

Each borough is referenced by its 4-digit Local Authority code (since superseded by [GSS codes](https://en.wikipedia.org/wiki/ONS_coding_system#Current_GSS_coding_system)). To map, we will need to link the location code to the Borough name and grid location, which we can do in a separate location table just as we did with the simpler examples above:

^^^elm{m=(tableSummary 6 locationTable)}^^^

And then join to the tidied OD table:

```elm {l=hidden}
odTable : Table
odTable =
    let
        compactLocationTable =
            locationTable
                |> removeColumn "GSS"
                |> removeColumn "Name"
                |> removeColumn "longitude"
                |> removeColumn "latitude"

        t1 =
            leftJoin ( tidyOd, "o" ) ( compactLocationTable, "LACode" )
                |> removeColumn "LACode"
                |> removeColumn "o"
                |> renameColumn "ShortName" "from"
                |> renameColumn "row" "oRow"
                |> renameColumn "col" "oCol"
    in
    leftJoin ( t1, "d" ) ( compactLocationTable, "LACode" )
        |> removeColumn "LACode"
        |> removeColumn "d"
        |> renameColumn "ShortName" "to"
        |> renameColumn "row" "dRow"
        |> renameColumn "col" "dCol"
```

^^^elm{m=(tableSummary 6 odTable)}^^^

Now we can map the data as we did with the simpler examples:

```elm {l=hidden}
odMapFull : String -> String -> Spec
odMapFull large small =
    let
        cfg =
            configure
                << configuration (coHeader [ hdLabelFontSize 0, hdTitle "" ])
                << configuration (coFacet [ facoSpacing 5 ])
                << configuration (coView [ vicoStroke Nothing ])

        flowData =
            dataFromColumns []
                << dataColumn "oCol" (nums (numColumn "oCol" odTable))
                << dataColumn "oRow" (nums (numColumn "oRow" odTable))
                << dataColumn "dCol" (nums (numColumn "dCol" odTable))
                << dataColumn "dRow" (nums (numColumn "dRow" odTable))
                << dataColumn "o" (strs (strColumn "from" odTable))
                << dataColumn "d" (strs (strColumn "to" odTable))
                << dataColumn "weight" (strs (strColumn "numJourneys" odTable))

        trans =
            transform
                << calculateAs ("slice(datum." ++ large ++ ",0,1)") "initial"

        enc =
            encoding
                << position X [ pName (small ++ "Col"), pAxis [] ]
                << position Y [ pName (small ++ "Row"), pAxis [] ]
                << color
                    [ mName "weight"
                    , mQuant
                    , mScale [ scType scPow, scExponent 0.4 ]
                    , mTitle "Number of journeys"
                    ]

        cellSpec =
            asSpec
                [ enc []
                , square [ maSize 130, maOpacity 1, maStrokeWidth 1.5 ]
                ]

        homeTrans =
            transform
                << filter (fiExpr "datum.o == datum.d")

        homeEnc =
            encoding
                << position Y [ pName (large ++ "Row"), pOrdinal, pAxis [] ]
                << position X [ pName (large ++ "Col"), pOrdinal, pAxis [] ]
                << text [ tName "initial" ]

        homeSpec =
            asSpec [ homeTrans [], homeEnc [], textMark [ maStroke "#fff", maFontSize 6 ] ]

        odSpec =
            asSpec [ width 100, height (100 * 7 / 8), layer [ cellSpec, homeSpec ] ]
    in
    toVegaLite
        [ cfg []
        , title ("Large cells: " ++ large ++ ", small cells: " ++ small) []
        , flowData []
        , trans []
        , facet
            [ rowBy [ fName (large ++ "Row") ]
            , columnBy [ fName (large ++ "Col") ]
            ]
        , specification odSpec
        ]
```

^^^elm {v=(odMapFull "o" "d")}^^^

^^^elm {v=(odMapFull "d" "o")}^^^

{(task|}

Looking at the two OD maps of London commuting, what patterns can you detect? Remember the labelled cells represent 'home' locations, where origin and destination are the same borough. Longest journeys are therefore those that are furthest away from the home cell.

{|task)}

As a further example of how this may be applied to real data, see Robert Radburn's, [Virtual Water OD Map](https://public.tableau.com/profile/robradburn#!/vizhome/LiquidCapital/VirtualWater) showing the network of water transfer across the US in the form of agricultural production and distribution:

![OD map approach](https://staff.city.ac.uk/~jwo/datavis2023b/session09/images/odmap2.png)

## 5. Conclusions

Networks are an important category of data relationships and their visualization allows us to make relational comparisons between them. Because networks are often very complex, there is a danger that poor visualizations simply show that complexity without revealing any structure or insight into the data being represented. You should therefore select carefully the form of data aggregation and representation you use when showing networks.

The examples shown in this session should provide you with some design approaches for constructing your own network visualizations. But be aware that commonly used, node-link diagrams do not scale well to larger more complex networks. Non node-link designs like hive plots and matrix views offer alternatives that can scale more effectively, especially if the number of edges is high compared to the number of nodes (i.e. the network is highly connected). Matrix views remove the co-linearity and overlapping nodes and edges problem, but are not as effective in showing sub-graphs within the network. Which form of representation you choose will depend (as ever) on your audience and the characteristics of the network you wish to emphasise.

## 6. Recommended Reading

To become good at data visualization you will need to read beyond the lecture notes. 'Recommended reading' points you to accessible sources of further information related to the content of each session. At the very minimum, please read the relevant chapter of Munzner (2015).

**Munzner, T.** (2015) Chapter 9: _Arrange Networks and Trees_, pp.200-217 in [Visualization Analysis and Design](https://go.exlibris.link/9jMy6fQG), CRC Press

### References

**Beecham, R. and Wood, J.** (2014). [Exploring gendered cycling behaviours within a large-scale behavioural dataset.](https://openaccess.city.ac.uk/14154/) _Transportation Planning and Technology_, 37(1), pp. 83-97

**Krzywinski, M, Birol, I., Jones, J. and Marra, M.** (2012) [Hive plots - rational approach to visualizing networks.](https://academic.oup.com/bib/article/13/5/627/412507) _Briefings in Bioinformatics_, 13(5), pp.627-644

**Wood, J., Dykes, J. and Slingsby, A.** (2010) [Visualisation of origins, destinations and flows with OD Maps](http://openaccess.city.ac.uk/537/). _Cartographic Journal, 47(2)_, pp. 117-129.

## 7. Practical Exercises

{(task|} Please add answers to the following to your portfolio repo for session 9. This is the last of the continuous assessment exercises and is shorter than the others so you have time to devote to working on your datavis project, remembering to push regularly to the `project` folder of your repo. {|task)}

### 1. Comparison of Network Visualization Approaches

Considering the following techniques mentioned in the lecture, identify at least one advantage and disadvantage of each:

| Visualization Type             | Advantages         | Disadvantages      |
| ------------------------------ | ------------------ | ------------------ |
| Node-link diagram              | _your answer here_ | _your answer here_ |
| Matrix view                    | _your answer here_ | _your answer here_ |
| Sankey diagram                 | _your answer here_ | _your answer here_ |
| OD Map (grid map of grid maps) | _your answer here_ | _your answer here_ |

### 2. Co-authorship Insights

Reviewing the various representations of the giCentre co-authorship visualizations, identify the three most useful insights you've gained. Briefly identify which visualization(s) led to the insight and how it did.

- Insight 1: _your answer here_
- Insight 2: _your answer here_
- Insight 3: _your answer here_

---

## Appendices

### Appendix A : Table representing giCentre publications

_See source code of this document for details._

```elm {l=hidden}
journalData : Table
journalData =
    """Name1,Name2,Name3,Name4,Name5,Name6,Name7,Name8,Name9,Date,Title,id
Youssef A,Reyes Aldasoro C,,,,,,,,2022,Computational Image Analysis Techniques Programming Languages and Software Platforms Used in Cancer Research: A Scoping Review,28535
Andrienko N,Andrienko G,Chen S,Fisher B,,,,,,2022,Seeking Patterns of Visual Pattern Discovery for Knowledge Building,28386
Sun Y,Li J,Chen S,Andrienko G,Andrienko N,Zhang K,,,,2022,A learning-based approach for efficient visualization construction,28154
Bunks C,Weyde T,Slingsby A,Wood J,,,,,,2022,Visualization of Tonal Harmony for Jazz Lead Sheets,28140
Dykes J,Jianu R,Slingsby A,Bach B,Borgo R,Chen M,Enright J,Fang H,Wood J,2022,Visualization for Epidemiological Modelling: Challenges Solutions Reflections & Recommendations,28123
Chen M,Jianu R,Archambault D,Dykes J,Slingsby A,Ritsos P,Wood J,Turkay C,Bach B,2022,RAMPVIS: Answering the Challenges of Building Visualisation Capabilities for Large-scale Emergency Responses,28077
Andrienko G,Andrienko N,Cordero Garcia J,Hecker D,Vouros G,,,,,2022,Supporting Visual Exploration of Iterative Job Scheduling,28062
Andrienko N,Andrienko G,Adilova L,Wrobel S,Rhyne T-M,,,,,2022,Visual Analytics for Human-Centered Machine Learning,27550
Chow B,Reyes Aldasoro C,,,,,,,,2021,Automatic Gemstone Classification Using Computer Vision,27360
D'amato E,Reyes Aldasoro C,Consiglio A,D'amato G,Faienza M,Zollino M,,,,2021,Detection of Pitt‚Äö√Ñ√¨Hopkins syndrome based on morphological facial features,27359
Tominski C,Andrienko G,Andrienko N,Bleisch S,Fabrikant S,Mayr E,Miksch S,Pohl M,Skupin A,2021,Toward flexible visual analytics augmented through smooth display transitions,26795
Nanni M,Andrienko G,Barabási A-L,Boldrini C,Bonchi F,Cattuto C,Chiaromonte F,Comandé G,Conti M,2021,Give more data awareness and control to individual citizens and they will help COVID-19 containment.,26562
Ananda A,Ngan K,Karabağ C,Ter-Sarkisov A,Alonso E,Reyes Aldasoro C,,,,2021,Classification and Visualisation of Normal and Abnormal Radiographs: a comparison between Eleven Convolutional Neural Network Architectures,26534
Madabhushi A,Reyes Aldasoro C,,,,,,,,2021,Special Issue on Computational Pathology: An Overview,26486
Haider J,Pohl M,Beecham R,Dykes J,,,,,,2021,Strategies for Detecting Difference in Map Line-up Tasks,26314
Karabağ C,Jones M,Reyes Aldasoro C,,,,,,,2021,Volumetric Semantic Instance Segmentation of the Plasma Membrane of HeLa Cells,26302
Morgades P,Slingsby A,Moat J,,,,,,,2021,Assessing the Geographical Structure of Species Richness Data with Interactive Graphics,26289
Beecham R,Dykes J,Hama L,Lomax N,,,,,,2021,On the Use of 'Glyphmaps' for Analysing the Scale and Temporal Spread of COVID-19 Reported Cases,26275
Jaffari R,Hashmani M,Reyes Aldasoro C,,,,,,,2021,A Novel Focal Phi Loss for Power Line Segmentation with Auxiliary Classifier U-Net,26126
Andrienko G,Andrienko N,Kureshi I,Lee K,Smith I,Staykova T,,,,2021,Automating and utilising equal-distribution data classification,25464
Bonamy M,Bonnet E,Bousquet N,Charbit P,Giannopoulos P,Kim E,Rzazewski P,Sikora F,Thomasse S,2021,EPTAS and Subexponential Algorithm for Maximum Clique on Disk and Unit Ball Graphs,25413
Beecham R,Dykes J,Rooney C,Wong W,,,,,,2021,Design Exposition Discussion Documents for Rich Design Discourse in Applied Visualization,23913
Andrienko G,Andrienko N,Anzer G,Bauer P,Budziak G,Fuchs G,Hecker D,Weber H,Wrobel S,2021,Constructing Spaces and Times for Tactical Analysis in Football,23209
Ananda A,Karabağ C,Ter-Sarkisov A,Alonso E,Reyes Aldasoro C,,,,,2020,Radiography Classification: A comparison between Eleven Convolutional Neural Networks,27296
Liu F,Andrienko G,Andrienko N,Chen S,Janssens D,Wets G,Theodoridis Y,,,2020,Citywide Traffic Analysis Based on the Combination of Visual and Analytic Approaches,26087
Ngan K,Garcez A,Knapp K,Appelboam A,Reyes Aldasoro C,,,,,2020,A machine learning approach for Colles' fracture treatment diagnosis,25605
Reece J,Couvillon M,Grüter C,Ratnieks F,Reyes Aldasoro C,,,,,2020,Automatic Analysis of Bees‚Äö√Ñ√¥ Waggle Dance,25602
Andrienko N,Andrienko G,Miksch S,Schumann H,Wrobel S,,,,,2020,A theoretical model for pattern discovery in visual analytics,25463
Liem J,Perin C,Wood J,,,,,,,2020,Structure and Empathy in Visual Data Storytelling: Evaluating their Influence on Attitude,25322
Bianconi F,Kather J,Reyes Aldasoro C,,,,,,,2020,Experimental Assessment of Color Deconvolution and Color Normalization for Automated Classification of Histology Images Stained with Hematoxylin and Eosin,25289
Chen S,Andrienko N,Andrienko G,Li J,Yuan X,,,,,2020,Co-Bridges: Pair-wise Visual Connection and Comparison for Multi-item Data Streams,25207
Mitchell C,Caroff L,Solis-Lemus J,Reyes Aldasoro C,Vigilante A,Warburton F,de Chaumont F,Dufour A,Dallongeville S,2020,Cell Tracking Profiler: a user-driven analysis framework for evaluating 4D live cell imaging data,25147
Karabağ C,Jones M,Peddie C,Weston A,Collinson L,Reyes Aldasoro C,,,,2020,Semantic segmentation of HeLa cells: An objective comparison between one traditional algorithm and four deep-learning architectures,25013
Ortega-Ruiz M,Karabağ C,Garduño V,Reyes Aldasoro C,,,,,,2020,Morphological Estimation of Cellularity on Neo-Adjuvant Treated Breast Cancer Histological Images,25011
Reyes Aldasoro C,Ngan K,Ananda A,Garcez A,Appelboam A,Knapp K,,,,2020,Geometric semi-automatic analysis of radiographs of Colles‚Äö√Ñ√¥ fractures,24924
Abrahamsen M,Giannopoulos P,Löffler M,Rote G,,,,,,2020,Geometric Multicut: Shortest Fences for Separating Groups of Objects in the Plane,24746
Andrienko G,Andrienko N,Boldrini C,Caldarelli G,Cintia P,Cresci S,Facchini A,Giannotti F,Gionis A,2020,(So) Big Data and the transformation of the city,24627
Sîrbu A,Andrienko G,Andrienko N,Boldrini C,Conti M,Giannotti F,Guidotti R,Bertoli S,Kim J,2020,Human migration: the big data perspective,24626
Stiff T,Echegaray-Iturra F,Pink H,Herbert A,Reyes Aldasoro C,Hochegger H,,,,2020,Prophase-Specific Perinuclear Actin Coordinates Centrosome Separation and Positioning to Ensure Accurate Chromosome Segregation,24394
Solis-Lemus J,Sánchez-Sánchez B,Marcotti S,Burki M,Stramer B,Reyes Aldasoro C,,,,2020,Comparative Study of Contact Repulsion in Control and Mutant Macrophages Using a Novel Interaction Detection,24274
Karabağ C,Jones M,Peddie C,Weston A,Collinson L,Reyes Aldasoro C,,,,2020,Geometric differences between nuclear envelopes of Wild-type and Chlamydia trachomatis-infected HeLa cells,24221
Ortega-Ruiz M,Karabağ C,Garduño V,Reyes Aldasoro C,,,,,,2020,Morphological estimation of Cellularity on Neo-adjuvant treated breast cancer histological images,24220
Ceneda D,Andrienko N,Andrienko G,Gschwandtner T,Miksch S,Piccolotto N,Schreck T,Streit M,Suschnigg J,2020,Guide Me in Analysis: A Framework for Guidance Designers,24217
Brown V,Turkay C,Jianu R,,,,,,,2020,Dissecting Visual Analytics: Comparing Frameworks for Interpreting and Modelling Observed Visual Analytics Behavior,24171
Solis-Lemus J,Sánchez-Sánchez B,Marcotti S,Burki M,Stramer B,Reyes Aldasoro C,,,,2020,Comparison of Interactions Between Control and Mutant Macrophages,23849
Meyer M,Dykes J,,,,,,,,2020,Criteria for Rigor in Visualization Design Study,22644
Hall K,Bradley A,Hinrichs U,Huron S,Wood J,Collins C,Carpendale S,,,2020,Design by immersion: A transdisciplinary approach to problem-driven visualizations,22491
Giannopoulos P,Abrahamsen M,Löffler M,Rote G,,,,,,2020,Geometric Multicut,22319
Chen S,Li J,Andrienko G,Andrienko N,Wang Y,Nguyen P,Turkay C,,,2020,Supporting Story Synthesis: Bridging the Gap between Visual Analytics and Storytelling,21217
Chen S,Andrienko G,Andrienko N,Doulkeridis C,Koumparos A,,,,,2019,Contextualized Analysis of Movement Events,27570
Ilczyszyn A,Ringrose C,Goldsworthy N,Beresineviciute L,Reyes Aldasoro C,Appelboam A,Knapp K,,,2019,Colles‚Äö√Ñ√¥ Fractures: Intra- and Inter-operator precision of alignment measurements from projection radiographs pre- and post-manipulation under anaesthesia.,25607
Reyes Aldasoro C,,,,,,,,,2019,Redesign of the assessment materials for the Module EE3517 Medical Imaging Modalities,25606
Chen S,Chen S,Andrienko N,Andrienko G,Nguyen P,Turkay C,Thonnard O,Yuan X,,2019,User Behavior Map: Visual Exploration for Cyber Security Session Data,23211
Vouros G,Santipantakis G,Doulkeridis C,Vlachou A,Andrienko G,Andrienko N,Fuchs G,Cordero Garcia J,Martinez M,2019,The datAcron Ontology for the Specification of Semantic Trajectories,23210
Nouri J,Vasilakos I,Yan Y,Reyes Aldasoro C,,,,,,2019,Effect of Viscosity and Speed on Oil Cavitation Development in a Single Piston-Ring Lubricant Assembly,23008
Karabağ C,Jones M,Peddie C,Weston A,Collinson L,Reyes Aldasoro C,,,,2019,Segmentation and modelling of hela nuclear envelope,22914
Karabağ C,Jones M,Peddie C,Weston A,Collinson L,Reyes Aldasoro C,,,,2019,Segmentation and Modelling of the Nuclear Envelope of HeLa Cells Imaged with Serial Block Face Scanning Electron Microscopy,22871
Karabağ C,Verhoeven J,Miller N,Reyes Aldasoro C,,,,,,2019,Texture Segmentation: An Objective Comparison between Traditional and Deep-Learning Methodologies,22870
Pazhakh V,Ellett F,Croker B,O'Donnell J,Pase L,Schulze K,Greulich R,Gupta A,Reyes Aldasoro C,2019,≈í‚â§-glucan-dependent shuttling of conidia from neutrophils to macrophages occurs during fungal infection establishment,22869
Karabağ C,Verhoeven J,Miller N,Reyes Aldasoro C,,,,,,2019,Texture Segmentation: An Objective Comparison between Five Traditional Algorithms and a Deep-Learning U-Net Architecture,22853
Verhoeven J,Marien P,De Clerck I,Daems L,Reyes Aldasoro C,Miller N,,,,2019,Asymmetries in speech articulation as reflected on palatograms: a meta-study,22692
Miller N,Reyes Aldasoro C,Verhoeven J,,,,,,,2019,Asymmetries in tongue-palate contact during speech,22691
Bonnet E,Giannopoulos P,,,,,,,,2019,Orthogonal Terrain Guarding is NP-complete,22638
Nguyen P,Henkin R,Chen S,Andrienko N,Andrienko G,Thonnard O,Turkay C,,,2019,VASABI: Hierarchical User Profiles for Interactive Visual User Behaviour Analytics,22591
Andrienko N,Andrienko G,Patterson F,Stange H,,,,,,2019,Visual Analysis of Place Connectedness by Public Transport,22511
Bianconi F,Kather J,Reyes Aldasoro C,,,,,,,2019,Evaluation of Colour Pre-processing on Patch-Based Classification of H&E-Stained Images,22489
Slingsby A,Hyde J,Turkay C,,,,,,,2019,Visual Analysis of Reactionary Train Delay from an Agent Based Model,22460
Giannopoulos P,Bonnet E,Lampis M,,,,,,,2019,On the Parameterized Complexity of Red-Blue Points Separation,22279
Beecham R,Slingsby A,,,,,,,,2019,Characterising labour market self-containment in London with geographically arranged small multiples,22242
Allain K,Turkay C,Dykes J,,,,,,,2019,Towards a WHAT-WHY-HOW Taxonomy of Trajectories in Visualization Research,22236
Solis-Lemus J,Stramer B,Slabaugh G,Reyes Aldasoro C,,,,,,2019,MacroSight: A novel framework to analyze the shape and movement of interacting macrophages using MAtLABR ‚Äö√Ñ‚Ä†,22139
Katner K,Jianu R,,,,,,,,2019,The Effectiveness of Nudging in Commercial Settings and Impact on User Trust,21946
Silva N,Blascheck T,Jianu R,Rodriguez N,Raubal M,Schreck T,Weiskopf D,,,2019,Eye Tracking Support for Visual Analytics Systems,21945
Chen S,Andrienko N,Andrienko G,Adilova L,Barlet J,Kindermann J,Nguyen P,Thonnard O,Turkay C,2019,LDA Ensembles for Interactive Exploration and Categorization of Behaviors,21875
Verhoeven J,Miller N,Daems L,Reyes Aldasoro C,,,,,,2019,Visualisation and Analysis of Speech Production with Electropalatography,21872
Demsar U,Slingsby A,Weibel R,,,,,,,2019,Introduction to the special section on Visual Movement Analytics,21831
Kather J,Krisam J,Charoentong P,Luedde T,Herpel E,Weis C,Gaiser T,Marx A,Reyes Aldasoro C,2019,Predicting survival from colorectal cancer histology slides using deep learning: A retrospective multicenter study,21373
Andrienko N,Andrienko G,Garcia J,Scarlatti D,,,,,,2019,Analysis of Flight Variability: a Systematic Approach,20334
Dykes J,Kerzner E,Goodwin S,Jones S,Meyer M,,,,,2019,A Framework for Creative Visualization-Opportunities Workshops,20085
Wood J,Kachkaev A,Dykes J,,,,,,,2019,Design Exposition with Literate Visualization,20081
Leandrou S,Mamais I,Petroudi S,Kyriacou P,Reyes Aldasoro C,Pattichis C,,,,2018,Hippocampal and entorhinal cortex volume changes in Alzheimer's disease patients and mild cognitive impairment subjects.,26190
Andrienko G,Andrienko N,Fuchs G,Scarlatti D,Cordero Garcia J,Vouros G,Herranz R,Marcos R,,2018,Visual analytics of flight trajectories for uncovering decision making strategies,23796
Tampakis P,Pelekis N,Theodoridis Y,Andrienko N,Andrienko G,Fuchs G,,,,2018,Time-aware Sub-Trajectory Clustering in Hermes@PostgreSQL,23212
Vouros G,Doulkeridis C,Santipantakis G,Vlachou A,Pelekis N,Georgiou H,Theodoridis Y,Andrienko G,Andrienko N,2018,Big data analytics for time critical maritime and aerial mobility forecasting,22887
Sathiyanarayanan M,Turkay C,Dykes J,,,,,,,2018,Visualising E-mail Communication to Improve E-discovery,22850
Collins C,Andrienko N,Schreck T,Yang J,Choo J,Engelke U,Jena A,Dwyer T,,2018,Guidance in the human‚Äö√Ñ√¨machine analytics process,22367
Liu S,Andrienko G,Wu Y,Cao N,Jiang L,Shi C,Wang Y,Hong S,,2018,Steering data quality with visual analytics: The complexity challenge,22366
Meyer M,Dykes J,,,,,,,,2018,Reflection on Reflection in Applied Visualization Research Generating Knowledge From Practice,21833
Kanai J,Grant R,Jianu R,,,,,,,2018,Cities on and off the map: A bibliometric assessment of urban globalisation research,21512
Alt H,Cabello S,Giannopoulos P,Knauer C,,,,,,2018,Minimum Cell Connection in Line Segment Arrangements,21510
Giannopoulos P,Bonnet E,Kim E,Rzazewski P,Sikora F,,,,,2018,QPTAS and Subexponential Algorithm for Maximum Clique on Disk Graphs,21507
Giannopoulos P,Bonnet E,,,,,,,,2018,Orthogonal Terrain Guarding is NP-complete,21500
Slabaugh G,Reyes Aldasoro C,,,,,,,,2018,Guest Editorial: Computer Vision in Cancer Data Analysis,21375
Dunkel A,Andrienko G,Andrienko N,Burghardt D,Hauthal E,Purves R,,,,2018,A conceptual framework for studying collective reactions to events in location-based social media,21218
Li J,Chen S,Chen W,Andrienko G,Andrienko N,,,,,2018,Semantics-Space-Time Cube. A Conceptual Framework for Systematic Analysis of Texts in Space and Time,21109
Slingsby A,,,,,,,,,2018,Tilemaps for Summarising Multivariate Geographical Variation,20884
Slingsby A,Paterson A,Rigby M,Grace K,Nguyen P,Perin C,Turkay C,Aljasem D,,2018,Characterising Farms by the Movement of Animals through Them,20883
Jawaid M,Narejo S,Pirzada N,Baloch J,Reyes Aldasoro C,Slabaugh G,,,,2018,Automated quantification of non-calcified coronary plaques in cardiac CT angiographic imagery,20871
Olliverre N,Yang G,Slabaugh G,Reyes Aldasoro C,Alonso E,,,,,2018,Generating Magnetic Resonance Spectroscopy Imaging Data of Brain Tumours from Linear Non-Linear and Deep Learning Models.,20508
Okoe M,Jianu R,Kobourov S,,,,,,,2018,Node-link or Adjacency Matrices: Old Question New Insights,20159
Andrienko G,Andrienko N,,,,,,,,2018,Creating maps of artificial spaces to explore trajectories,20132
Li J,Chen S,Zhang K,Andrienko G,Andrienko N,,,,,2018,COPE: Interactive Exploration of Co-occurrence Patterns in Spatial Time Series.,20131
Nguyen P,Turkay C,Andrienko G,Andrienko N,Thonnard O,Zouaoui J,,,,2018,Understanding User Behaviour through Action Sequences: from the Usual to the Unusual,20108
Reyes Aldasoro C,,,,,,,,,2018,Shape Analysis and Tracking of Migrating Macrophages,20035
Vouros G,Vlachou A,Santipantakis G,Doulkeridis C,Pelekis N,Georgiou H,Theodoridis Y,Andrienko G,Andrienko N,2018,Increasing maritime situation awareness via trajectory detection enrichment and recognition of events,20034
Solis-Lemus J,Stramer B,Slabaugh G,Reyes Aldasoro C,,,,,,2018,Shape analysis and tracking of migrating macrophages,20033
Markovic N,Sekula P,Vander Laan Z,Andrienko G,Andrienko N,,,,,2018,Applications of Trajectory Data From the Perspective of a Road Transportation Agency: Literature Review and Maryland Case Study,19987
Perin C,Vuillemot R,Stolper C,Stasko J,Wood J,Carpendale S,,,,2018,State of the Art of Sports Data Visualization,19857
Karabağ C,Jones M,Peddie C,Weston A,Collinson L,Reyes Aldasoro C,,,,2018,Automated Segmentation of HeLa Nuclear Envelope from Electron Microscopy Images,19564
Miller N,Verhoeven J,Reyes Aldasoro C,,,,,,,2018,Analysis of the symmetry of electrodes for Electropalatography with Cone Beam CT Scanning,19563
Solis-Lemus J,Stramer B,Slabaugh G,Reyes Aldasoro C,,,,,,2018,Analysis of the Interactions of Migrating Macrophages,19562
Kather J,Berghoff A,Ferber D,Suarez Carmona M,Reyes Aldasoro C,Valous N,Rojas-Moraleda R,Jäger D,Halama N,2018,Large-scale database mining reveals hidden trends and future directions for cancer immunotherapy,19561
Okoe M,Jianu R,Kobourov S,,,,,,,2018,Revisited experimental comparison of node-link and matrix representations,19215
Andrienko N,Lammarsch T,Andrienko G,Fuchs G,Keim D,Miksch S,Rind A,,,2018,Viewing Visual Analytics as Model Building,19078
Beecham R,Slingsby A,Brunsdon C,,,,,,,2018,Locally-varying explanations behind the United Kingdom‚Äö√Ñ√¥s vote to the Leave the European Union,19048
Leandrou S,Petroudi S,Reyes Aldasoro C,Kyriacou P,Pattichis C,,,,,2018,Quantitative MRI Brain Studies in Mild Cognitive Impairment and Alzheimer's disease: A Methodological Review,18891
Stein M,Janetzko H,Lamprecht A,Breitkreutz T,Zimmermann P,Goldlucke B,Schreck T,Andrienko G,Grossniklaus M,2018,Bring it to the Pitch: Combining Video and Movement Data to Enhance Team Sport Analysis,18380
Andrienko G,Andrienko N,Fuchs G,Cordero Garcia J,,,,,,2018,Clustering Trajectories by Relevant Parts for Air Traffic Analysis,18119
Vrotsou K,Fuchs G,Andrienko N,Andrienko G,,,,,,2017,An Interactive Approach for Exploration of Flows Through Direction-Based Filtering,26366
Giannopoulos P,Bonnet E,Lampis M,,,,,,,2017,On the Parameterized Complexity of Red-Blue Points Separation,21508
Claramunt C,Ray C,Salmon L,Camossi E,Hadzagic M,Jousselme A-L,Andrienko G,Andrienko N,Theodoridis Y,2017,Maritime data integration and analysis: Recent progress and research challenges,19854
Rooney C,Beecham R,Dykes J,Wong W,,,,,,2017,Dynamic Design Documents for supporting applied visualization,19479
Solis-Lemus J,Stramer B,Slabaugh G,Reyes Aldasoro C,,,,,,2017,Segmentation and Shape Analysis of Macrophages Using Anglegram Analysis,19349
Reyes Aldasoro C,,,,,,,,,2017,The proportion of cancer related entries in PubMed has increased considerably,18496
Andrienko G,Andrienko N,Weibel R,,,,,,,2017,Geographic data science,18379
Reyes Aldasoro C,Slabaugh G,,,,,,,,2017,Special issue: medical image understanding and analysis,18325
Wood J,,,,,,,,,2017,Visual Analytic Design for Detecting Airborne Pollution Sources,18289
Slingsby A,van Loon E,,,,,,,,2017,Visual Characterisation of Temporal Occupancy for Movement Ecology,18270
Jawaid M,Riaz A,Rajani R,Reyes Aldasoro C,Slabaugh G,,,,,2017,Framework for Detection and Localization of Coronary Non-Calcified Plaques in Cardiac CTA using Mean Radial Profiles,17951
Reyes Aldasoro C,Björndahl M,Kanthou C,Tozer G,,,,,,2017,Topological analysis of the vasculature of angiopoietin-expressing tumours through scale-space tracing,17944
Solis-Lemus J,Stramer B,Slabaugh G,Reyes Aldasoro C,,,,,,2017,Segmentation of Overlapping Macrophages Using Anglegram Analysis,17830
Jawaid M,Rajani R,Liatsis P,Reyes Aldasoro C,Slabaugh G,,,,,2017,Improved CTA Coronary Segmentation with a Volume-Specific Intensity Threshold,17829
Shurkhovetskyy G,Andrienko N,Andrienko G,Fuchs G,,,,,,2017,Data Abstraction for Visualizing Large Time Series,17818
Sacha D,Al-Masoudi F,Steinbrecher M,Schreck T,Keim D,Andrienko G,Janetzko H,,,2017,Dynamic Visual Abstraction of Soccer Movement,17817
Laird A,Riedel M,Okoe M,Jianu R,Ray K,Eickhoff S,Smith S,Fox P,Sutherland M,2017,Heterogeneous fractionation profiles of meta-analytic coactivation networks,17747
Andrienko G,Andrienko N,Budziak G,Dykes J,Fuchs G,Von Landesberger T,Weber H,,,2017,Visual Analysis of Pressure in Football,17464
Andrienko N,Andrienko G,Camossi E,Claramunt C,Cordero Garcia J,Fuchs G,Hadzagic M,Jousselme A-L,Ray C,2017,Visual exploration of movement and event data with interactive time masks,17427
Beecham R,Slingsby A,Brunsdon C,Radburn R,,,,,,2017,Spatially varying explanations behind the UKs vote to leave the EU,17399
Slingsby A,van Loon E,,,,,,,,2017,Temporal tile-maps for characterising the temporal occupancy of places: A seabird case study,17398
Nguyen P,Turkay C,Andrienko G,Andrienko N,Thonnard O,,,,,2017,A Visual Analytics Approach for User Behaviour Understanding through Action Sequence Analysis,17284
Reyes Aldasoro C,,,,,,,,,2017,The proportion of cancer-related entries in PubMed has increased considerably; is cancer truly 'The Emperor of All Maladies'?,17241
Andrienko N,Andrienko G,,,,,,,,2017,State Transition Graphs for Semantic Analysis of Movement Behaviours,17215
Andrienko G,Andrienko N,Chen W,Maciejewski R,Zhao Y,,,,,2017,Visual Analytics of Mobility and Transportation: State of the Art and Further Research Directions,17214
Jawaid M,Rajani R,Liatsis P,Reyes Aldasoro C,Slabaugh G,,,,,2017,A Hybrid Energy Model for Region Based Curve Evolution - Application to CTA Coronary Segmentation,17194
Jianu R,Alam S,,,,,,,,2017,A Data Model and Task Space for Data of Interest (DOI) Eye-Tracking Analyses,16941
Turkay C,Slingsby A,Lahtinen K,Butt S,Dykes J,,,,,2017,Supporting Theoretically-grounded Model Building in the Social Sciences through Interactive Visualisation,16153
Van Goethem A,Staals F,Löffler M,Dykes J,Speckmann B,,,,,2017,Multi-Granular Trend Detection for Time-Series Analysis,15761
Andrienko G,Andrienko N,Fuchs G,Wood J,,,,,,2017,Revealing Patterns and Trends of Mass Mobility through Spatial and Temporal Abstraction of Origin-Destination Movement Data,15488
Meulemans W,Dykes J,Slingsby A,Turkay C,Wood J,,,,,2017,Small Multiples with Gaps,15167
Beecham R,Dykes J,Meulemans W,Slingsby A,Turkay C,Wood J,,,,2017,Map LineUps: effects of spatial structure on graphical inference,15119
Pinheiro R,Pradhananga N,Jianu R,Orabi W,,,,,,2016,Eye-tracking technology for construction safety: A feasibility study,27577
Cabello S,Giannopoulos P,,,,,,,,2016,The Complexity of Separating Points in the Plane,21509
Ramirez Perez F,Pennington K,Pollock K,Esangbedo O,Foote C,Reyes Aldasoro C,Wu H-H,Ji T,Martinez-Lemus L,2016,Maternal Hyperleptinemia Increases Arterial Stiffening and Alters Vasodilatoy Responses to Insulin in Adult Male Mice Offspring,18315
Duckham M,van Kreveld M,Purves R,Speckmann B,Tao Y,Verbeek K,Wood J,,,2016,Modeling Checkpoint-Based Movement with the Earth Mover's Distance,17712
Green R,Simoes F,Reyes Aldasoro C,Rossor A,Scoto M,Barri M,Greensmith L,Muntoni F,Reilly M,2016,A DYNC1H1 mutation in autosomal dominant spinal muscular atrophy shows the potential of pharmacological inhibition of histone deacetylase 6 as a treatment for disease associated cellular phenotypes,15763
Fonseca J,Nadimi S,Reyes Aldasoro C,O’Sullivan C,Coop M,,,,,2016,Image-based investigation into the primary fabric of stress transmitting particles in sand,15707
Alam S,Jianu R,,,,,,,,2016,Analyzing Eye-Tracking Information in Visualization and Data Space: from Where on the Screen to What on the Screen.,15527
Wood J,,,,,,,,,2016,Visual Analytic Design for Contextualising Sensor Data,15523
Henkin R,Dykes J,Slingsby A,,,,,,,2016,Characterizing Representation of Temporal Data Visualization,15521
Andrienko N,Andrienko G,Rinzivillo S,,,,,,,2016,Leveraging spatial abstraction in traffic analysis and forecasting with visual analytics,15470
Andrienko G,Andrienko N,Budziak G,Von Landesberger T,Weber H,,,,,2016,Coordinate transformations for characterization and cluster analysis of spatial configurations in football,15469
Gomez S,Jianu R,Cabeen R,Guo H,Laidlaw D,,,,,2016,Fauxvea: Crowdsourcing Gaze Location Estimates for Visualization Analysis Tasks,15398
Cartas Ayala A,Aghaei M,Grüter C,Ratnieks F,Reyes Aldasoro C,,,,,2016,Behavior Analysis of Ants from Video Sequences,15309
McCurdy N,Dykes J,Meyer M,,,,,,,2016,Action Design Research and Visualization Design,15270
Foote C,Castorena-Gonzalez J,Ramirez Perez F,Jia G,Hill M,Reyes Aldasoro C,Sowers J,Martinez-Lemus L,,2016,Arterial stiffening in western diet-fed mice is associated with increased vascular elastin transforming growth factor-‚àö√º and plasma neuraminidase,15250
Andrienko G,Andrienko N,Dykes J,Kraak M,Robinson A,Schumann H,,,,2016,GeoVisual analytics: interactivity dynamics and scale,15025
Bouts Q,Dwyer T,Dykes J,Speckmann B,Goodwin S,Henry-Riche N,Carpendale S,Liebman A,,2016,Visual Encoding of Dissimilarity Data via Topology-Preserving Map Deformation,15023
Mason J,Klippel A,Bleisch S,Slingsby A,Deitrick S,,,,,2016,Special issue introduction: Approaching spatial uncertainty visualization to support reasoning and decision making,14922
Pennington K,Ramirez Perez F,Pollock K,Talton O,Foote C,Reyes Aldasoro C,Wu H-H,Ji T,Martinez-Lemus L,2016,Maternal hyperleptinemia is associated with male offspring‚Äö√Ñ√¥s altered vascular function and structure in mice,14796
Andrienko G,Andrienko N,Fuchs G,,,,,,,2016,Understanding movement data quality,14725
Beecham R,Rooney C,Meier S,Dykes J,Slingsby A,Turkay C,Wood J,Wong W,,2016,Faceted Views of Varying Emphasis (FaVVEs): a framework for visualising multi-perspective small multiples,14252
Turkay C,Slingsby A,Lahtinen K,Butt S,Dykes J,,,,,2016,Enhancing a Social Science Model-building Workflow with Interactive Visualisation,14232
Andrienko N,Andrienko G,Fuchs G,Jankowski P,,,,,,2016,Scalable and privacy-respectful interactive discovery of place semantics from human mobility traces,14193
Slingsby A,van Loon E,,,,,,,,2016,Exploratory Visual Analysis for Animal Movement Ecology,14159
Andrienko N,Andrienko G,Rinzivillo S,,,,,,,2016,Leveraging spatial abstraction in traffic analysis and forecasting with visual analytics,13599
Kather J,Weis C,Zöllner F,Reyes Aldasoro C,,,,,,2016,Mapping tumour tissue: quantitative maps of histological whole slide images,13486
Von Landesberger T,Brodkorb F,Roskosch P,Andrienko N,Andrienko G,Kerren A,,,,2016,MobilityGraphs: Visual Analysis of Mass Mobility Dynamics via Spatio-Temporal Graphs and Clustering,12849
Llado X,Imiya A,Mason D,Reyes Aldasoro C,Aoki K,Kudo M,Zhang Y,Argyriou V,,2015,Corrigendum to 'Homage to Professor Maria Petrou' [ Pattern Recognition Letters 48 (2014) 2-7].,17765
Ostermann F,Huang H,Andrienko G,Andrienko N,Capineri C,Farkas K,Purves R,,,2015,Extracting and comparing places using geo-social media,15407
Okoe M,Jianu R,,,,,,,,2015,GraphUnit: Evaluating Interactive Graph Visualizations Using Crowdsourcing,15380
Leandrou S,Petroudi S,Kyriacou P,Reyes Aldasoro C,Pattichis C,,,,,2015,An overview of quantitative magnetic resonance imaging analysis studies in the assessment of alzheimer‚Äö√Ñ√¥s disease,14805
Vrotsou K,Janetzko H,Navarra C,Fuchs G,Spretke D,Mansmann F,Andrienko N,Andrienko G,,2015,SimpliFly: A Methodology for Simplification and Thematic Enhancement of Trajectories.,14164
Andrienko N,Andrienko G,Fuchs G,Jankowski P,,,,,,2015,Visual Analytics Methodology for Scalable and Privacy-Respectful Discovery of Place Semantics from Episodic Mobility Data,13025
Andrienko G,Andrienko N,,,,,,,,2015,Visualization Support to Interactive Cluster Analysis,13024
Andrienko N,Andrienko G,Fuchs G,Rinzivillo S,Betz H-D,,,,,2015,Real Time Detection and Tracking of Spatial Event Clusters,13023
Fonseca J,Reyes Aldasoro C,Wils L,,,,,,,2015,Three-dimensional quantification of the morphology and intragranular void ratio of a shelly carbonate sand,13009
Reyes Aldasoro C,Barri M,Hafezparast M,,,,,,,2015,Automatic segmentation of focal adhesions from mouse embryonic fibroblasts,12811
Andrienko N,Andrienko G,Rinzivillo S,,,,,,,2015,Exploiting spatial abstraction in predictive analytics of vehicle traffic,12466
Solis-Lemus J,Huang Y,Wlodkovic D,Reyes Aldasoro C,,,,,,2015,Microfluidic environment and tracking analysis for the observation of Artemia Franciscana,12421
Wood J,,,,,,,,,2015,Visualizing Personal Progress in Participatory Sports Cycling Events,12351
Goodwin S,Dykes J,Slingsby A,Turkay C,,,,,,2015,Visualizing Multiple Variables Across Scale and Geography,12337
Vives S,Dykes J,Merryweather A,,,,,,,2015,Visualization for Equity Analysts: Using the DSM in Stock Picking,12335
Dykes J,Rooney C,Beecham R,Turkay C,Slingsby A,Wood J,Wong W,,,2015,Multi-Perspective Synopsis with Faceted Views of Varying Emphasis,12334
Lahtinen K,Slingsby A,Dykes J,Butt S,Fitzgerald R,,,,,2015,Informing Non-Response Bias Model Creation in Social Surveys with Visualisation,12333
Henkin R,Slingsby A,Dykes J,,,,,,,2015,Exploring Temporal Granularities with Visualization,12332
Beecham R,Dykes J,Slingsby A,Turkay C,,,,,,2015,Supporting crime analysis through visual design,12331
Slabaugh G,Reyes Aldasoro C,,,,,,,,2015,Guest Editorial: Medical Image Understanding and Analysis,12274
Huang Y,Reyes Aldasoro C,Persoone G,Wlodkowic D,,,,,,2015,Integrated microfluidic technology for sub-lethal and behavioral marine ecotoxicity biotests,12197
Kather J,Marx A,Reyes Aldasoro C,Schad L,Zöllner F,Weis C,,,,2015,Continuous representation of tumor microvessel density and detection of angiogenic hotspots in histological whole-slide images,12182
Bender S,Castorena-Gonzalez J,Garro M,Reyes Aldasoro C,Sowers J,DeMarco V,Martinez-Lemus L,,,2015,Regional variation in arterial stiffening and dysfunction in western diet-induced obesity,12181
Andrienko G,Andrienko N,Fuchs G,,,,,,,2015,Multi-perspective analysis of mobile phone call data records: A visual analytics approach,11979
Hoffmann E,Reyes Aldasoro C,,,,,,,,2015,Automatic segmentation of centromeres foci and delineation of chromosomes,11967
Mian A,Reyes Aldasoro C,,,,,,,,2015,Quantification of the Effects of Low Dose Radiation and its Impact on Cardiovascular Risks,11966
Blazakis K,Madzvamuse A,Reyes Aldasoro C,Styles V,Venkataraman C,,,,,2015,Whole cell tracking through the optimal control of geometric evolution laws,8689
Yin T,Ali F,Reyes Aldasoro C,,,,,,,2015,A Robust and Artifact Resistant Algorithm of Ultrawideband Imaging System for Breast Cancer Detection,8683
Fonseca J,Reyes Aldasoro C,O’Sullivan C,Coop M,,,,,,2015,Experimental investigation into the primary fabric of stress transmitting particles,4138
Dykes J,Bleisch S,,,,,,,,2015,Quantitative data graphics in 3D desktop-based virtual environments ‚Äö√Ñ√¨ an evaluation,3973
Wood J,,,,,,,,,2014,Visual analytics of GPS tracks: From location to place to behaviour,17705
Jianu R,Rusu A,Hu Y,Taggart D,,,,,,2014,How to Display Group Information on Node-Link Diagrams: An Evaluation,15379
Okoe M,Alam S,Jianu R,,,,,,,2014,A Gaze-enabled Graph Visualization to Improve Graph Reading Tasks,15378
Beecham R,Wood J,,,,,,,,2014,Exploring gendered cycling behaviours within a large-scale behavioural data-set,14154
Van Goethem A,Haverkort H,Meulemans W,Reimer A,Speckmann B,Wood J,,,,2014,Automatic schematization with curved lines,14152
Van Goethem A,Reimer A,Speckmann B,Wood J,,,,,,2014,Stenomaps: Shorthand for shapes,14151
Beecham R,Wood J,,,,,,,,2014,Towards confirmatory data analysis? Deriving and analysing routing information for an origin-destination bike share dataset,13003
Andrienko N,Andrienko G,Fuchs G,Stange H,,,,,,2014,Detecting and tracking dynamic clusters of spatial events,11978
Andrienko N,Andrienko G,Fuchs G,,,,,,,2014,Analysis of mobility behaviors in geographic and semantic spaces,11977
Slingsby A,Kelly M,Dykes J,,,,,,,2014,Featured graphic. OD maps for showing changes in Irish female migration between 1851 and 1911,8483
Van Goethem A,Meulemans W,Speckmann B,Wood J,,,,,,2014,Exploring curved schematization,8221
Blazakis K,Reyes Aldasoro C,Styles V,Venkataraman C,Madzvamuse A,,,,,2014,An optimal control approach to cell tracking,6556
Barthet M,Plumbley M,Kachkaev A,Dykes J,Wolff D,Weyde T,,,,2014,Big Chord Data Extraction and Mining,5803
Kachkaev A,Wolff D,Barthet M,Tidhar D,Plumbley M,Dykes J,Weyde T,,,2014,Visualising Chord Progressions in Music Collections: A Big Data Approach,5629
Williams L,Mukherjee D,Fisher M,Reyes Aldasoro C,Akerman S,Kanthou C,Tozer G,,,2014,An in vivo role for Rho kinase activation in the tumour vascular disrupting activity of combretastatin A-4 3-O-phosphate,5496
Tozer G,Daniel R,Lunt S,Reyes Aldasoro C,Cunningham V,,,,,2014,Haemodynamics and Oxygenation of the Tumor Microcirculation,5485
Llado X,Imiya A,Reyes Aldasoro C,Mason D,Aoki K,Kudo M,Zhang Y,Argyriou V,,2014,Homage to Professor Maria Petrou,5484
Kachkaev A,Wood J,,,,,,,,2014,Automated planning of leisure walks based on crowd-sourced photographic content,4943
Beecham R,Dykes J,Turkay C,Slingsby A,Wood J,,,,,2014,Map Line Ups: Using Graphical Inference to Study Spatial Structure,4783
Henkin R,Slingsby A,van Loon E,,,,,,,2014,Designing Interactive Graphics for Foraging Trips Exploration,4774
Henkin R,Kachkaev A,Slingsby A,,,,,,,2014,Summarising the structure of an organisation and reconstructing a chain of events - VAST 2014 Mini-Challenge 1 Submission Honourable Mention for Novelty in Visualization,4773
Kanthou C,Dachs G,Lefley D,Steele A,Coralli-Foxon C,Harris S,Greco O,Dos Santos S,Reyes Aldasoro C,2014,Tumour cells expressing single VEGF isoforms display distinct growth survival and migration characteristics,4142
Slingsby A,Tate N,Fisher P,,,,,,,2014,Visualisation of Uncertainty in a Geodemographic Classifier,4119
Turkay C,Slingsby A,Hauser H,Wood J,Dykes J,,,,,2014,Interactive Generation of Visual Summaries for Multivariate Geographical Data Analysis,4117
Fisher P,Tate N,Slingsby A,,,,,,,2014,Type-2 Fuzzy Sets Applied to Geodemographic Classification,4115
Weyde T,Cottrell SJ,Dykes J,Benetos E,Wolff D,Tidhar D,Gold N,Abdallah S,Plumbley M,2014,Big Data for Musicology,4077
Bowyer S,Kanthou C,Reyes Aldasoro C,,,,,,,2014,Analysis of capillary-like structures formed by endothelial cells in a novel organotypic assay developed from heart tissue,3953
Goodwin S,Dykes J,Slingsby A,,,,,,,2014,Visualizing the Effects of Scale and Geography in Multivariate Comparison,3876
Turkay C,Slingsby A,Hauser H,Wood J,Dykes J,,,,,2014,Attribute Signatures: Dynamic Visual Summaries for Analyzing Multivariate Geographical Data,3867
Wood J,Beecham R,Dykes J,,,,,,,2014,Moving beyond sequential design: Reflections on a rich multi-channel approach to data visualization,3839
Wood J,Badawood D,,,,,,,,2014,The Effect of Information Visualization Delivery on Narrative Construction and Development,3702
Slingsby A,Dykes J,Wood J,Radburn R,,,,,,2014,Designing an Exploratory Visual Interface to the Results of Citizen Surveys,3649
Beecham R,Wood J,,,,,,,,2014,Characterising group-cycling journeys using interactive graphics,3439
Kachkaev A,Wood J,Dykes J,,,,,,,2014,Glyphs for exploring crowd-sourced subjective survey classification,3393
Beecham R,Wood J,Bowerman A,,,,,,,2014,Studying commuting behaviours using collaborative visual analytics,2871
Jianu R,Laidlaw D,,,,,,,,2013,What Google Maps can do for biomedical data dissemination: examples and a design study,15376
Reyes Aldasoro C,,,,,,,,,2013,Cancer Image Analysis,8690
Kanthou C,Dachs G,Lefley D,Reyes Aldasoro C,English W,,,,,2013,Endogenous VEGF isoform expression regulates tumour cell motility,8373
Akerman S,Fisher M,Daniel R,Lefley D,Reyes Aldasoro C,Lunt S,Harris S,Björndahl M,Williams L,2013,Influence of soluble or matrix-bound isoforms of vascular endothelial growth factor-A on tumor response to vascular-targeted strategies,5498
Andrienko N,Andrienko G,Barrett L,Dostie M,Henzi P,,,,,2013,Space transformation for understanding group movement,4326
Henry K,Pase L,Ramos-Lopez C,Lieschke G,Renshaw S,Reyes Aldasoro C,,,,2013,PhagoSight: an open-source MATLAB¬¨√Ü package for the analysis of fluorescent neutrophil and macrophage migration in a zebrafish model,3735
Andrienko N,Andrienko G,,,,,,,,2013,Exploring Traffic in Brest Harbor by Trajectory Aggregation,2912
Andrienko G,Andrienko N,,,,,,,,2013,Exploring Trajectory Attributes in Brest Harbor,2911
Fuchs G,Andrienko N,Andrienko G,Bothe S,Stange H,,,,,2013,Tracing the German Centennial Flood in the Stream of Tweets: First Lessons Learned,2909
Andrienko N,Andrienko G,Fuchs G,,,,,,,2013,Towards Privacy-Preserving Semantic Mobility Analysis,2908
Andrienko G,Andrienko N,Fuchs G,Olteanu Raimond A,Symanzik J,Ziemlicki C,,,,2013,Extracting Semantics of Individual Places from Movement Data by Analyzing Temporal Patterns of Visits,2907
Andrienko G,Andrienko N,,,,,,,,2013,Spatio-temporal analysis of flows in CDC 2013 data,2905
Fuchs G,Andrienko G,Andrienko N,Jankowski P,,,,,,2013,Extracting Personal Behavioral Patterns from Geo-Referenced Tweets,2904
Andrienko N,Andrienko G,,,,,,,,2013,Visual analytics of movement: An overview of methods tools and procedures,2856
Andrienko G,Andrienko N,Hurter C,Rinzivillo S,Wrobel S,,,,,2013,Scalable analysis of movement data for extracting and exploring significant places,2852
Andrienko N,Andrienko G,,,,,,,,2013,A visual analytics framework for spatio-temporal analysis and modelling,2851
Kachkaev A,Wood J,,,,,,,,2013,Investigating Spatial Patterns in User-Generated Photographic Datasets by Means of Interactive Visual Analytics,2829
Kachkaev A,Wood J,,,,,,,,2013,Crowd-sourced Photographic Content for Urban Recreational Route Planning,2828
Badawood D,Wood J,,,,,,,,2013,A visual language to characterise transitions in narrative visualization,2817
Dove G,Jones S,Dykes J,Brown A,Duffy A,,,,,2013,Using data visualization in creativity workshops: a new tool in the designer's kit,2814
Kachkaev A,Wood J,,,,,,,,2013,Exploring Subjective Survey Classification of a Photographic Archive using Visual Analytics,2785
Walker R,Slingsby A,Dykes J,Xu K,Wood J,Nguyen P,Stephens D,Wong W,Zheng Y,2013,An Extensible Framework for Provenance in Human Terrain Visual Analytics,2619
Goodwin S,Dykes J,Jones S,Dillingham I,Dove G,Duffy A,Kachkaev A,Slingsby A,Wood J,2013,Creative User-Centered Visualization Design for Energy Analysts and Modelers,2618
Ali R,Dykes J,Wood J,,,,,,,2013,Framework for Studying Spatially Ordered Treemaps,2609
Slingsby A,Beecham R,Wood J,,,,,,,2013,Visual analysis of social networks in space and time using smartphone logs,2530
Slingsby A,Radburn R,,,,,,,,2013,Green Spaces: Interactively Mapping the Results of a Public Consultation,2387
Slingsby A,van Loon E,,,,,,,,2013,Visual Analytics for Exploring Changes in Biodiversity,2385
Kölzsch A,Slingsby A,Wood J,Nolet B,Dykes J,,,,,2013,Visualisation design for representing bird migration tracks in time and space,2384
Zhang Q,Slingsby A,Dykes J,Wood J,Kraak M,Blok C,Ahas R,,,2013,Visual analysis design to support research into movement and use of space in Tallinn: A case study,2383
Dillingham I,Dykes J,Wood J,,,,,,,2013,Visualizing the Geographies of the Haiti Crisis Map,2124
Dillingham I,Wood J,Dykes J,,,,,,,2013,Types Granularities and Combinations of Geographic Objects in the Haiti Crisis Map,2123
Kelly M,Slingsby A,Dykes J,Wood J,,,,,,2013,Historical Internal Migration in Ireland,2052
Kachkaev A,Dillingham I,Beecham R,Goodwin S,Ahmed N,Slingsby A,,,,2013,Monitoring the Health of Computer Networks with Visualization - VAST 2012 Mini Challenge 1 Award: 'Efficient Use of Visualization',1324
Slingsby A,Beecham R,Wood J,,,,,,,2013,Visual analysis of social networks in space and time using smartphone logs,1236
Andrienko N,Andrienko G,Stange H,Liebig T,Hecker D,,,,,2012,Visual Analytics for Understanding Spatial Situations from Episodic Movement Data,13256
Kanthou C,Ghareai Z,Haagen J,Lunt S,Reyes Aldasoro C,Doeer W,Tozer G,,,2012,Inhibition of angiogenesis in the mouse heart by ionizing radiation,8374
Henry K,Reyes Aldasoro C,Renshaw S,,,,,,,2012,Exploring neutrophil behaviour in a zebrafish model of inflammation through the generation of novel parameters using MatLab algorithms,5763
Holmes G,Anderson S,Dixon G,Robertson A,Reyes Aldasoro C,Billings S,Renshaw S,Kadirkamanathan V,,2012,Repelled from the wound or randomly dispersed? Reverse migration behaviour of neutrophils characterized by dynamic modelling,5505
Griffiths M,Reyes Aldasoro C,Rodenberg J,,,,,,,2012,A Framework for Providing Research Applications as a Service Using the IOME Toolkit,4640
Pase L,Layton J,Wittmann C,Ellett F,Nowell C,Reyes Aldasoro C,Varma S,Rogers K,Hall C,2012,Neutrophil-delivered myeloperoxidase dampens the hydrogen peroxide burst after tissue wounding in zebrafish,4590
Reyes Aldasoro C,Henry K,Renshaw S,,,,,,,2012,Tracking neutrophils in zebrafish: the use of synthetic data sets,4317
Bhalerao A,Pase L,Lieschke G,Renshaw S,Reyes Aldasoro C,,,,,2012,Local affine texture tracking for serial registration of zebrafish images,4307
Kadirkamanathan V,Anderson S,Billings S,Zhang X,Holmes G,Reyes Aldasoro C,Elks P,Renshaw S,,2012,The neutrophil's eye-view: inference and visualisation of the chemoattractant field driving cell chemotaxis in vivo,3738
Holmes G,Dixon G,Anderson S,Reyes Aldasoro C,Elks P,Billings S,Whyte M,Kadirkamanathan V,Renshaw S,2012,Drift-Diffusion Analysis of Neutrophil Migration during Inflammation Resolution in a Zebrafish Model,3737
Reyes Aldasoro C,Björndahl M,Akerman S,Ibrahim J,Griffiths M,Tozer G,,,,2012,Online chromatic and scale-space microvessel-tracing analysis for transmitted light optical images,3736
Andrienko G,Andrienko N,Burch M,Weiskopf D,,,,,,2012,Visual analytics methodology for eye movement studies,2855
Tominski C,Schumann H,Andrienko G,Andrienko N,,,,,,2012,Stacking-based visualization of trajectory attribute data,2854
Slingsby A,Dykes J,,,,,,,,2012,Experiences in involving analysts in visualisation design,2136
Badawood D,Wood J,,,,,,,,2012,Effects of candidate position on ballot papers: Exploratory visualization of voter choice in the London local council elections 2010,1740
Beecham R,Wood J,Bowerman A,,,,,,,2012,Identifying and explaining inter-peak cycling behaviours within the London Cycle Hire Scheme Conference,1621
Badawood D,Wood J,,,,,,,,2012,The effect of information visualization delivery on narrative construction and development,1576
Kachkaev A,Wood J,,,,,,,,2012,Using Visual Analytics to Detect Problems in Datasets Collected From Photo-Sharing Services,1320
Slingsby A,Kelly M,Dykes J,Wood J,,,,,,2012,OD Maps for Studying Historical Internal Migration in Ireland,1312
Beecham R,Wood J,Bowerman A,,,,,,,2012,A visual analytics approach to understanding cycling behaviour,1299
Goodwin S,Dykes J,,,,,,,,2012,Visualising Variations in Household Energy Consumption,1294
Dillingham I,Dykes J,Wood J,,,,,,,2012,A Design Analysis and Evaluation Model to Support the Visualization Designer-User,1281
Wood J,Isenberg P,Isenberg T,Dykes J,Boukhelifa N,Slingsby A,,,,2012,Sketchy rendering for information visualization,1274
Dillingham I,Dykes J,Wood J,,,,,,,2012,Exploring Patterns of Uncertainty in Crowdsourced Crisis Information,1127
Goodwin S,Dykes J,,,,,,,,2012,Geovisualization of household energy consumption characteristics,895
Dillingham I,Dykes J,Wood J,,,,,,,2012,Characterising Locality Descriptions in Crowdsourced Crisis Information,892
Lunt S,Akerman S,Hill S,Fisher M,Wright V,Reyes Aldasoro C,Tozer G,Kanthou C,,2011,Vascular effects dominate solid tumor response to treatment with combretastatin A-4-phosphate,28092
Ortega-Ruiz M,Karabağ C,Garduño V,Reyes Aldasoro C,,,,,,2011,Estimation of cellularity in tumours treated with Neoadjuvant therapy: A comparison of Machine Learning algorithms,24222
Andrienko G,Andrienko N,Mladenov M,Mock M,P‚àö‚àÇlitz C,,,,,2011,Identifying Place Histories from Activity Traces with an Eye to Parameter Impact.,7615
Williams L,Fisher M,Reyes Aldasoro C,Kanthou C,Tozer G,,,,,2011,Abstract 1578: A critical role for RhoA-GTPase signaling in the tumor vascular disrupting action of combretastatin-A4-phosphate in vivo,5945
Tozer G,Akerman S,Cross N,Barber P,Reyes Aldasoro C,Greco O,Harris S,Hill S,Honess D,2011,Blood vessel maturation and response to vascular-disrupting therapy in single vascular endothelial growth factor-A isoform-producing tumors,5509
Reyes Aldasoro C,Williams L,Akerman S,Kanthou C,Tozer G,,,,,2011,An automatic algorithm for the segmentation and morphological analysis of microvessels in immunostained histological tumour sections,5508
Elks P,van Eeden F J,Dixon G,Wang X,Reyes Aldasoro C,Ingham P,Whyte M,Walmsley S,Renshaw S,2011,Activation of hypoxia-inducible factor-1≈í¬± (Hif-1≈í¬±) delays inflammation resolution by reducing neutrophil apoptosis and reverse migration in a zebrafish inflammation model,5507
Reyes Aldasoro C,Björndahl M,Akerman S,Ibrahim J,Tozer G,,,,,2011,An on-line chromatic and scale-space microvasculature-tracing analysis for transmitted light optical images,5442
Akerman S,Reyes Aldasoro C,Fisher M,Pettyjohn K,Björndahl M,Evans H,Tozer G,,,2011,Microflow of fluorescently labelled red blood cells in tumours expressing single isoforms of VEGF and their response to vascular targeting agents,3740
Andrienko G,Andrienko N,Keim D,MacEachren A,Wrobel S,,,,,2011,Challenging problems of geospatial visual analytics,2841
Andrienko G,Andrienko N,Heurich M,,,,,,,2011,An event-based conceptual model for context-aware movement analysis,2839
Andrienko G,Andrienko N,Bak P,Keim D,Kisilevich S,Wrobel S,,,,2011,A conceptual framework and taxonomy of techniques for analyzing movement,2837
Slingsby A,,,,,,,,,2011,Supporting the visual analysis of the behaviour of gulls,2528
Dillingham I,Mills B,Dykes J,,,,,,,2011,Exploring Road Incident Data with Heat Maps,738
Kornfeld A,Schiewe J,Dykes J,,,,,,,2011,Audio Cartography: Visual Encoding of Acoustic Parameters,616
Slingsby A,Wood J,Dykes J,,,,,,,2011,Visualizing Bicycle Hire Model Distributions,562
Slingsby A,Strachan J,Dykes J,Wood J,Vidale P,,,,,2011,Designing Interactive Graphics for Validating and Interpreting Storm Track Model Outputs,561
Koh L,Slingsby A,Dykes J,Kam T,,,,,,2011,Developing and applying a user-centered model for the design and implementation of information visualization tools,555
Wood J,Slingsby A,Dykes J,,,,,,,2011,Visualizing the dynamics of London's bicycle hire scheme,538
Purves R,Edwardes A,Wood J,,,,,,,2011,Describing place through user generated content,535
Fekete J,Hemery P,Baudel T,Wood J,,,,,,2011,Obvious: a meta-toolkit to encapsulate information visualization toolkits. One toolkit to bind them all,534
Dillingham I,Dykes J,Wood J,,,,,,,2011,Visual Analytical Approaches to Evaluating Uncertainty and Bias in Crowdsourced Crisis Information,466
Ali R,Dykes J,,,,,,,,2011,ESRI vs BREWER: An Evaluation of Map Use with Alternative Colour Schemes amongst the General Public,440
Lloyd D,Dykes J,,,,,,,,2011,Human-Centered Approaches in Geovisualization Design: Investigating Multiple Methods Through a Long-Term Case Study,438
Slingsby A,Dykes J,Wood J,,,,,,,2011,Exploring Uncertainty in Geodemographics with Interactive Graphics,437
Wood J,Badawood D,Dykes J,Slingsby A,,,,,,2011,BallotMaps: Detecting name bias in alphabetically ordered ballot papers,436
Williams L,Fisher M,Reyes Aldasoro C,Kanthou C,Tozer G,,,,,2010,A critical role for RhoA-GTPase signaling in the tumour vascular disrupting action of combretastatin A4-phosphate in vivo,8372
Kanthou C,Reyes Aldasoro C,Tozer G,,,,,,,2010,Signaling interactions between RhoGTPase and cAMP/cGMP influence endothelial responses to the vascular disrupting agent combretastatin A4 phosphate,8371
Andrienko N,Andrienko G,,,,,,,,2010,Spatial generalization and aggregation of massive movement data.,7614
Reyes Aldasoro C,Björndahl M,Ibrahim J,Akerman S,Tozer G,,,,,2010,A scale-space tracing algorithm for analysis of tumour blood vessel morphology from transmitted light optical images,5441
Reyes Aldasoro C,Williams L,Akerman S,Kanthou C,Tozer G,,,,,2010,Segmentation and morphological analysis of microvessels in immunostained histological tumour sections,4644
Reyes Aldasoro C,Griffiths M,Savas D,Tozer G,,,,,,2010,CAIMAN: an online algorithm repository for Cancer Image Analysis,3739
Slingsby A,Strachan J,Vidale P,Dykes J,Wood J,,,,,2010,Making hurricane track data accessible,3028
Andrienko G,Andrienko N,Demsar U,Dransch D,Dykes J,Fabrikant S,Jern M,Kraak M,Schumann H,2010,Space time and visual analytics,2853
Andrienko G,Andrienko N,,,,,,,,2010,A general framework for using aggregation in visual exploration of movement data,2850
Andrienko G,Andrienko N,Bremm S,Schreck T,Von Landesberger T,Bak P,Keim D,,,2010,Space-in-time and time-in-space self-organizing maps for exploring spatiotemporal patterns,2847
Andrienko G,Andrienko N,Bak P,Bremm S,Keim D,Von Landesberger T,P‚àö‚àÇlitz C,Schreck T,,2010,A framework for using self-organising maps to analyse spatiotemporal patterns exemplified by analysis of mobile phone usage,2838
Radburn R,Beecham R,Dykes J,Wood J,Slingsby A,,,,,2010,Discovery exhibition: using spatial treemaps in local authority decision making and reporting,1300
Wood J,Dykes J,Slingsby A,,,,,,,2010,Visualisation of Origins Destinations and Flows with OD Maps,537
Slingsby A,Wood J,Dykes J,,,,,,,2010,Treemap Cartography for showing Spatial and Temporal Traffic Patterns,419
Slingsby A,Dykes J,Wood J,,,,,,,2010,Rectangular Hierarchical Cartograms for Socio-Economic Data,418
Wood J,Radburn R,Dykes J,,,,,,,2010,vizLib: Using The Seven Stages of Visualization to Explore Population Trends and Processes in Local Authority Research,416
Clarke J,Dykes J,Hemsley-Flint F,Medyckyj-Scott D,Sietinsone L,Slingsby A,Urwin T,Wood J,,2010,vizLegends : Re-Imagining Map Legends with Visualization,415
Slingsby A,Wood J,Dykes J,Clouston D,Foote M,,,,,2010,Visual analysis of sensitivity in CAT models: interactive visualisation for CAT model sensitivity analysis,413
Slingsby A,Dykes J,Wood J,Radburn R,,,,,,2010,OAC Explorer: Interactive exploration and comparison of multivariate socioeconomic population characteristics,410
Wood J,Slingsby A,Dykes J,,,,,,,2010,Layout and colour transformations for visualizing OAC data,407
Wood J,Slingsby A,Dykes J,,,,,,,2010,Designing visual analytics systems for disease spread and evolution: VAST 2010 mini challenge 2 and 3 award: Good overall design and analysis,404
Dykes J,Wood J,Slingsby A,,,,,,,2010,Rethinking Map Legends with Visualization,178
Griffiths M,Reyes Aldasoro C,Savas D,Greenfield T,,,,,,2009,IOME A Toolkit for Distributed and Collaborative Computational Science and Engineering,4649
Reyes Aldasoro C,Zhao Y,Coca D,Billings S,Kadirkamanathan V,Tozer G,Renshaw S,,,2009,Analysis of immune cell function using in vivo cell shape analysis and tracking,4647
Akerman S,Reyes Aldasoro C,Fisher M,Pettyjohn K,Björndahl M,Evans H,Tozer G,,,2009,Microflow of fluorescently labelled red blood cells in tumours expressing single isoforms of VEGF and their response to VEGF-R tyrosine kinase inhibition,4645
Reyes Aldasoro C,,,,,,,,,2009,Retrospective shading correction algorithm based on signal envelope estimation,3741
Wood J,Slingsby A,Khalili-Shavarini N,Dykes J,Mountain D,,,,,2009,Visualization of uncertainty and analysis of geographical data,414
Dykes J,Lloyd D,Radburn R,,,,,,,2009,Using the Analytic Hierarchy Process to prioritise candidate improvements to a geovisualization application,412
Khalili N,Wood J,Dykes J,,,,,,,2009,Mapping the geography of social networks,408
Wood J,Dykes J,Slingsby A,Radburn R,,,,,,2009,Flow trees for exploring spatial trajectories,405
Bleisch S,Dykes J,Nebiker S,,,,,,,2009,Building bridges between methodological approaches: a meta-framework linking experiments and applied studies in 3D geovisualization research,403
Slingsby A,Lowe R,Dykes J,Stephenson D,Wood J,Jupp T,,,,2009,A pilot study for the collaborative development of new ways of visualising seasonal climate forecasts,402
Slingsby A,Dykes J,Wood J,,,,,,,2009,Configuring Hierarchical Layouts to Address Research Questions,158
Williams L,Reyes Aldasoro C,Kanthou C,Tozer G,,,,,,2008,THE CONTRIBUTION OF THE RHO PROTEIN PATHWAY TO THE TUMOUR VASCULAR DISRUPTING ACTION OF CA-4-P,25608
Reyes Aldasoro C,Akerman S,Tozer G,,,,,,,2008,Measuring the velocity of fluorescently labelled red blood cells with a keyhole tracking algorithm.,8688
Reyes Aldasoro C,Wilson I,Prise V,Barber P,Ameer-Beg M,Vojnovic B,Cunningham V,Tozer G,,2008,Estimation of apparent tumor vascular permeability from multiphoton fluorescence microscopic images of P22 rat sarcomas in vivo,5510
Reyes Aldasoro C,Biram D,Tozer G,Kanthou C,,,,,,2008,Measuring cellular migration with image processing,4311
Rinzivillo S,Pedreschi D,Nanni M,Giannotti F,Andrienko N,Andrienko G,,,,2008,Visually driven analysis of movement data by progressive clustering,2849
Andrienko G,Andrienko N,Bartling U,,,,,,,2008,Visual analytics approach to user-controlled evacuation scheduling,2848
Slingsby A,Raper J,,,,,,,,2008,Navigable Space in 3D City Models for Pedestrians,564
Slingsby A,Dykes J,Wood J,Foote M,Blom M,,,,,2008,The Visual Exploration of Insurance Data in Google Earth,563
Wood J,Dykes J,,,,,,,,2008,Spatially Ordered Treemaps,536
Arrell K,Wise S,Wood J,Donoghue D,,,,,,2008,Spectral filtering as a method of visualising and removing striped artefacts in digital elevation data,533
Goodwin S,Dykes J,,,,,,,,2008,Do Local Information Systems Hide the Bigger Picture? An analytical approach to measuring the strength of local boundaries,441
Lloyd D,Dykes J,Radburn R,,,,,,,2008,Mediating geovisualization to potential users and prototyping a geovisualization application,409
Slingsby A,Dykes J,Wood J,,,,,,,2008,Using treemaps for variable selection in spatio-temporal visualisation,177
Andrienko G,Andrienko N,Dykes J,Fabrikant S,Wachowicz M,,,,,2008,Geovisualization of dynamics movement and change: key issues and developing approaches in visualization research,175
Bleisch S,Dykes J,Nebiker S,,,,,,,2008,Evaluating the effectiveness of representing numeric information through abstract graphics in 3D desktop virtual environments,173
Reyes Aldasoro C,Akerman S,Tozer G,,,,,,,2007,Red Blood Cell Tracking and Velocity Measurement with a Keyhole Model of Movement,25609
Reyes Aldasoro C,Akerman S,Tozer G,,,,,,,2007,Measuring Red Blood Cell Velocity with a Keyhole Tracking Algorithm,4650
Reyes Aldasoro C,Bhalerao A,,,,,,,,2007,Volumetric texture segmentation by discriminant feature selection and multiresolution classification,4320
Dykes J,Brunsdon C,,,,,,,,2007,Geographically weighted visualization: Interactive graphics for scale-varying exploratory analysis,3227
Andrienko N,Andrienko G,,,,,,,,2007,Intelligent visualisation and information presentation for civil crisis management,2845
Andrienko G,Andrienko N,Jankowski P,Keim D,Kraak M,MacEachren A,Wrobel S,,,2007,Geovisual analytics for spatial decision support: Setting the research agenda,2843
Andrienko N,Andrienko G,,,,,,,,2007,Designing visual analytics methods for massive collections of movement data,2842
Slingsby A,,,,,,,,,2007,A Layer-Based Data Model as a Basis for Structuring 3D Geometrical Built-Environment Data with Poorly-Specified heights in a GIS Context,565
Dykes J,Lloyd D,Radburn R,,,,,,,2007,Understanding geovisualization users and their requirements: a user-centred approach,411
Slingsby A,Dykes J,Wood J,Clarke K,,,,,,2007,Mashup cartography: cartographic issues of using Google Earth for tag maps,400
Slingsby A,Dykes J,Wood J,Clarke K,,,,,,2007,Interactive tag maps and tag clouds for the multiscale exploration of large spatio-temporal datasets,399
Wood J,Dykes J,Slingsby A,Clarke K,,,,,,2007,Interactive visual exploration of a large spatio-temporal dataset: Reflections on a geovisualization mashup,176
Reyes Aldasoro C,Akerman S,Tozer G,,,,,,,2006,Measurement of Fluorescently Labelled Red Blood Cell Velocity Using Self-Organising Maps,25610
Reyes Aldasoro C,Bhalerao A,,,,,,,,2006,The Bhattacharyya space for feature selection and its application to texture segmentation,4313
Andrienko G,Andrienko N,Fischer R,Mues V,Schuck A,,,,,2006,Reactions to geovisualization: An experience from a European project,2846
Slingsby A,Longley P,,,,,,,,2006,A Conceptual Framework for Describing Microscale Pedestrian Access in the Built Environment,566
Lloyd D,Dykes J,,,,,,,,2006,Investigating Catchment Area Anomalies for a North England Store,406
Dykes J,Bleisch S,,,,,,,,2006,Planning Hikes Virtually: How Useful are Web-based 3D Visualizations?,401
Reyes Aldasoro C,Wilson I,Prise V,Barber P,Ameer-Beg S,Vojnovic B,Tozer G,,,2005,Measurement of Vascular Permeability from Multiphoton Microscopy of Experimental Tumours,25613
Reyes Aldasoro C,Wilson I,Ameer-Beg S,Vojnovic B,Tozer G,,,,,2005,Analysis of the effects of tumour vascular targeting drugs on the vascular permeability of experimental tumours through multiphoton microscopy.,25611
Chertov O,Komarov A,Mikhailov A,Andrienko G,Andrienko N,Gatalsky P,,,,2005,Geovisualization of forest simulation modelling results: A case study of carbon sequestration and biodiversity,2844
Andrienko G,Andrienko N,,,,,,,,2005,Blending aggregation and selection: Adapting parallel coordinates for the visualization of large datasets,2840
Reyes Aldasoro C,,,,,,,,,2004,Multiresolution Volumetric Texture Segmentation,25614
Slingsby A,Longley P,Parker C,,,,,,,2004,Aspects of the Design of a Three-Dimensional National Mapping Data Framework,567
Reyes Aldasoro C,Bhalerao A,,,,,,,,2003,Volumetric texture description and discriminant feature selection for MRI,4315
Slingsby A,,,,,,,,,2003,An Object-Orientated Approach to Hydrological Modelling using Triangular Irregular Networks,568
Dykes J,Mountain D,,,,,,,,2002,What I did on my vacation: spatio-temporal log analysis with interactive graphics and morphometric surface derivative,417
Reyes Aldasoro C,Aldeco A,,,,,,,,2000,Image Segmentation and Compression using Neural Networks,25603
Reyes Aldasoro C,,,,,,,,,1994,Algorithm to Compute Reduced Costs on a Graph,25086"""
        |> fromCSV
```

### Appendix B : Function to find all pairwise combinations of authors in an input table

_See source code of this document for details._

```elm {l=hidden}
authorPairs : Table -> Table
authorPairs inputTable =
    let
        transpose xss =
            let
                numCols =
                    List.head >> Maybe.withDefault [] >> List.length
            in
            List.foldr (List.map2 (::)) (List.repeat (numCols xss) []) xss

        addToFreqTable item freqTable =
            if Dict.member item freqTable then
                Dict.update item (Maybe.map ((+) 1)) freqTable

            else
                Dict.insert item 1 freqTable

        nameColumn n =
            strColumn ("Name" ++ String.fromInt n) inputTable

        orderTuple ( a, b ) =
            if a < b then
                ( a, b )

            else
                ( b, a )

        pairs =
            List.range 1 9
                |> List.map nameColumn
                |> transpose
                |> List.map (List.filter (Basics.not << String.isEmpty))
                |> List.concatMap pairwiseCombinations
                |> List.map orderTuple
                |> List.foldl addToFreqTable Dict.empty
    in
    empty
        |> insertColumn "author1" (List.map Tuple.first (Dict.keys pairs))
        |> insertColumn "author2" (List.map Tuple.second (Dict.keys pairs))
        |> insertColumn "freq" (List.map String.fromInt (Dict.values pairs))


pairwiseCombinations : List a -> List ( a, a )
pairwiseCombinations =
    let
        toTuple list =
            case list of
                [ s1, s2 ] ->
                    Just ( s1, s2 )

                _ ->
                    Nothing

        combinations k items =
            if k <= 0 then
                [ [] ]

            else
                case items of
                    [] ->
                        []

                    hd :: tl ->
                        let
                            appendedToAll item list =
                                List.map ((::) item) list
                        in
                        appendedToAll hd (combinations (k - 1) tl) ++ combinations k tl
    in
    combinations 2 >> List.filterMap toTuple


authorNames : Table -> Table
authorNames inputTable =
    let
        names =
            (strColumn "author1" (authorPairs inputTable) ++ strColumn "author2" (authorPairs inputTable)) |> Set.fromList |> Set.toList

        numCoAuthors author =
            (filterRows "author1" ((==) author) (authorPairs inputTable) |> strColumn "author2" |> List.length)
                + (filterRows "author2" ((==) author) (authorPairs inputTable) |> strColumn "author1" |> List.length)
                |> String.fromInt

        numPapers author =
            let
                colHead n =
                    "Name" ++ String.fromInt n

                numRows n total =
                    total + (filterRows (colHead n) ((==) author) journalData |> strColumn (colHead n) |> List.length)
            in
            List.foldl numRows 0 (List.range 1 9) |> String.fromInt
    in
    empty
        |> insertColumn "author" names
        |> insertColumn "numCoAuthors" (List.map numCoAuthors names)
        |> insertColumn "numPapers" (List.map numPapers names)
        |> insertColumn "rx" (randomIntsBetween ( 0, 999 ) 12345 (List.length names) |> List.map String.fromInt)
        |> insertColumn "ry" (randomIntsBetween ( 0, 999 ) 54321 (List.length names) |> List.map String.fromInt)
```

### Appendix C : Random number generation

_See source code of this document for details._

```elm {l=hidden}
randomFloatsBetween : ( Float, Float ) -> Int -> Int -> List Float
randomFloatsBetween ( minVal, maxVal ) seed numVals =
    randomFloats seed numVals
        |> List.map (\x -> x * (maxVal - minVal) + minVal)


randomIntsBetween : ( Float, Float ) -> Int -> Int -> List Int
randomIntsBetween ( minVal, maxVal ) seed numVals =
    randomFloats seed numVals
        |> List.map (\x -> round (x * (maxVal - minVal) + minVal))


randomFloats : Int -> Int -> List Float
randomFloats seed numVals =
    let
        iterate n initial fn =
            scanl (always fn) initial (List.range 1 n)

        scanl fn b =
            let
                scan a bs =
                    case bs of
                        hd :: tl ->
                            fn a hd :: bs

                        _ ->
                            []
            in
            List.foldl scan [ b ] >> List.reverse

        random ( s, _ ) =
            let
                ks =
                    { ia = 16807
                    , im = 2147483647
                    , am = 1 / 2147483647
                    , iq = 127773
                    , ir = 2836
                    , mask = 123459876
                    }

                idum =
                    Bitwise.xor s ks.mask

                idum2 =
                    ks.ia * (idum - (idum // ks.iq) * ks.iq) - ks.ir * (idum // ks.iq)
            in
            if idum2 < 0 then
                ( Bitwise.xor (idum2 + ks.im) ks.mask, ks.am * toFloat (idum2 + ks.im) )

            else
                ( Bitwise.xor idum2 ks.mask, ks.am * toFloat idum2 )
    in
    iterate numVals ( seed, 0 ) random
        |> List.map Tuple.second
        |> List.drop 1
```

### Appendix D: Author node table with positions

_See source code of this document for details._

```elm {l=hidden}
positionedNodeTable : Table
positionedNodeTable =
    """author,numCoAuthors,numPapers,x,y
Abdallah S,8,1,-284.63995,1129.1909
Abrahamsen M,3,2,-307.68976,1422.8907
Adilova L,10,2,385.95242,-301.82822
Aghaei M,4,1,-698.56323,-433.47592
Ahas R,6,1,521.7353,494.46216
Ahmed N,5,1,-49.302814,745.4834
Akerman S,21,15,-389.34137,106.930115
Al-Masoudi F,6,1,1242.2458,-438.04584
Alam S,2,3,1222.6144,102.503235
Aldeco A,1,1,-800.47906,218.05109
Ali F,2,1,-936.0113,427.32947
Ali R,2,2,909.7117,840.60376
Aljasem D,7,1,552.0872,247.1569
Allain K,2,1,43.058895,197.20847
Alonso E,8,3,-1104.0654,318.69748
Alt H,3,1,-546.5433,1378.2885
Ameer-Beg M,7,1,-397.5943,-145.62347
Ameer-Beg S,6,2,-667.4268,-99.573685
Ananda A,8,3,-1155.5378,82.38561
Anderson S,10,3,-380.94193,-277.507
Andrienko G,168,95,489.77252,-653.4268
Andrienko N,154,92,509.1715,-574.0462
Anzer G,8,1,630.47906,-400.01797
Aoki K,7,2,-1235.3397,-581.8461
Appelboam A,9,3,-1372.1288,29.019999
Archambault D,8,1,590.79645,406.12048
Argyriou V,7,2,-1187.6724,-678.7968
Arrell K,3,1,898.6303,1181.3165
Bach B,11,2,628.573,536.6142
Badawood D,3,5,193.10684,1103.3574
Bak P,9,3,820.91504,-507.20502
Baloch J,5,1,-983.9647,969.919
Barabási A-L,8,1,177.78224,-1465.3453
Barber P,14,3,-559.6959,2.4593797
Barlet J,8,1,190.9559,-222.90497
Barrett L,4,1,-3.556485,-1045.6411
Barri M,9,2,-1416.2369,312.41412
Barthet M,6,2,-214.19637,894.06647
Bartling U,2,1,35.57348,-1256.9324
Baudel T,3,1,797.48804,1242.51
Bauer P,8,1,437.9927,-411.93216
Beecham R,19,23,139.70955,668.00665
Bender S,6,1,-1180.088,-473.9412
Benetos E,8,1,-228.90224,1212.9575
Beresineviciute L,6,1,-1480.6823,82.64608
Berghoff A,8,1,-805.91113,-985.63354
Bertoli S,8,1,309.71765,-1115.819
Betz H-D,4,1,713.6596,-871.8747
Bhalerao A,4,4,-406.97556,-641.7834
Bianconi F,2,2,-703.1358,-939.0045
Billings S,13,4,-272.93985,-326.94125
Biram D,3,1,-194.67038,371.0799
Björndahl M,14,7,-327.8524,177.59402
Blascheck T,6,1,1347.635,-75.09376
Blazakis K,4,2,-1013.018,219.92932
Bleisch S,14,6,127.63415,121.25174
Blok C,6,1,417.60138,529.67053
Blom M,4,1,428.2004,1063.5753
Boldrini C,19,3,242.20679,-1359.4014
Bonamy M,8,1,-697.2252,1093.1659
Bonchi F,8,1,-12.981932,-1370.8234
Bonnet E,9,6,-833.082,1110.285
Borgo R,8,1,806.7916,636.9732
Bothe S,4,1,803.0043,-813.01715
Boukhelifa N,5,1,548.677,867.39667
Bousquet N,8,1,-590.8222,1118.8029
Bouts Q,7,1,787.52344,343.24716
Bowerman A,2,3,271.88586,1294.7402
Bowyer S,2,1,-252.4856,474.208
Bradley A,6,1,1176.974,472.37894
Breitkreutz T,8,1,1251.428,-557.69904
Bremm S,7,2,1076.9242,-406.11264
Brodkorb F,5,1,874.6253,-614.22974
Brown A,4,1,-9.147735,1184.7898
Brown V,2,1,904.26624,218.33125
Brunsdon C,4,3,-99.89215,890.39795
Budziak G,10,3,467.9068,-200.6707
Bunks C,3,1,373.40363,1307.9834
Burch M,3,1,1145.84,-485.32324
Burghardt D,5,1,777.55493,-5.1003394
Burki M,5,2,-879.1751,964.2019
Butt S,5,3,23.062387,534.9607
Cabeen R,4,1,1391.6149,-221.57207
Cabello S,3,2,-437.7446,1326.9583
Caldarelli G,8,1,559.81366,-1261.2603
Camossi E,10,2,309.10687,-845.52954
Cao N,7,1,-160.20549,-1211.6117
Capineri C,6,1,874.7526,-56.944626
Caroff L,8,1,-753.48456,725.7408
Carpendale S,17,3,920.8148,613.828
Cartas Ayala A,4,1,-801.1698,-355.9703
Castorena-Gonzalez J,10,2,-1107.899,-575.706
Cattuto C,8,1,-131.36482,-1438.8574
Ceneda D,8,1,847.83185,-297.2396
Charbit P,8,1,-755.92535,1284.7753
Charoentong P,8,1,-557.15594,-1331.8727
Chen M,11,2,696.04047,444.00714
Chen S,24,12,135.39462,-362.93018
Chen W,6,2,-31.113102,-473.59476
Chertov O,5,1,653.94543,-1182.3494
Chiaromonte F,8,1,74.67698,-1451.549
Choo J,7,1,1013.1145,-34.7363
Chow B,1,1,-1458.9768,-270.9919
Cintia P,8,1,453.06458,-1282.0553
Claramunt C,10,2,292.95743,-958.99243
Clarke J,7,1,613.0711,957.38727
Clarke K,3,3,773.63824,1047.6571
Clouston D,4,1,277.82697,1181.6064
Coca D,6,1,-96.280846,-198.1381
Collins C,13,2,992.99915,175.44975
Collinson L,5,5,-1163.4467,-280.7762
Comandé G,8,1,-30.880373,-1467.9209
Consiglio A,5,1,-971.831,-954.5501
Conti M,14,2,129.39124,-1331.4434
Coop M,4,2,-868.0877,511.9461
Coralli-Foxon C,8,1,-459.90176,413.76352
Cordero Garcia J,17,5,439.84775,-800.2426
Cottrell SJ,8,1,-346.12344,1051.7643
Couvillon M,4,1,-725.83966,-225.80211
Cresci S,8,1,553.40015,-1360.4722
Croker B,8,1,-380.4437,-876.822
Cross N,8,1,-620.2002,259.7816
Cunningham V,9,2,-464.4058,-60.16361
D'amato E,5,1,-929.1548,-859.67505
D'amato G,5,1,-1035.3567,-1041.6616
Dachs G,9,2,-462.4358,536.3015
Daems L,5,2,-899.7125,-110.76002
Dallongeville S,8,1,-613.8069,821.492
Daniel R,10,2,-449.11304,202.44402
De Clerck I,5,1,-962.34595,-197.90977
DeMarco V,6,1,-1118.3678,-384.80905
Deitrick S,4,1,-135.50732,537.7472
Demsar U,10,2,456.61374,121.33519
Dillingham I,11,9,86.18545,905.9268
Dixon G,13,3,-382.61575,-394.01337
Doeer W,6,1,-80.444,-71.10451
Donoghue D,3,1,1084.2833,1012.0246
Dos Santos S,8,1,-551.08075,455.8566
Dostie M,4,1,88.780075,-943.3095
Doulkeridis C,13,4,269.06528,-550.1875
Dove G,9,2,206.92368,914.09515
Dransch D,8,1,491.64215,16.777868
Duckham M,6,1,1021.823,594.4109
Duffy A,9,2,153.98799,821.2239
Dufour A,8,1,-597.3277,587.59186
Dunkel A,5,1,690.1256,-72.97336
Dwyer T,14,2,790.0058,206.83862
Dykes J,114,104,294.68,641.03894
Echegaray-Iturra F,5,1,-1287.3743,698.39966
Edwardes A,2,1,1116.9076,362.57358
Eickhoff S,8,1,1297.2156,181.76659
Elks P,13,3,-478.87903,-326.0272
Ellett F,14,2,-486.91705,-828.70416
Engelke U,7,1,1113.5758,-86.007515
English W,4,1,-319.37274,576.01526
Enright J,8,1,864.7451,724.0543
Esangbedo O,8,1,-886.9357,-437.83966
Evans H,6,2,-161.1397,30.579653
Fabrikant S,15,3,269.89105,-29.096107
Facchini A,8,1,446.8224,-1386.7457
Faienza M,5,1,-1052.1144,-872.65906
Fang H,8,1,699.32355,633.2935
Farkas K,6,1,956.98004,-133.72182
Fekete J,3,1,703.8676,1303.6453
Ferber D,8,1,-709.723,-1057.3643
Fischer R,4,1,102.81616,-1055.9589
Fisher B,3,1,18.285906,-582.8398
Fisher M,15,7,-213.79303,148.85832
Fisher P,2,2,-100.39965,1422.6711
Fitzgerald R,4,1,-147.96054,770.6649
Fonseca J,5,3,-775.208,419.85638
Foote C,13,3,-904.6405,-696.55316
Foote M,5,2,327.98834,1087.9669
Fox P,8,1,1349.0367,289.99704
Fuchs G,46,23,579.29297,-536.003
Gaiser T,8,1,-446.0384,-1335.7722
Garcez A,5,2,-1252.7935,-0.26428425
Garcia J,3,1,450.03152,-1095.2959
Garduño V,3,3,-1262.7592,-117.29979
Garro M,6,1,-1010.05426,-378.36392
Gatalsky P,5,1,759.64667,-1149.3125
Georgiou H,8,2,116.28514,-817.6107
Ghareai Z,6,1,-80.84223,120.03892
Giannopoulos P,15,10,-515.82794,1244.9825
Giannotti F,16,3,389.37988,-1200.2858
Gionis A,8,1,345.9771,-1411.1232
Gold N,8,1,-128.91664,1200.7285
Goldlucke B,8,1,1354.912,-561.66144
Goldsworthy N,6,1,-1376.6792,-76.62496
Gomez S,4,1,1440.7147,-317.5137
Goodwin S,19,9,409.88498,675.90344
Grace K,7,1,406.0005,295.3763
Grant R,2,1,1224.2352,830.8673
Greco O,14,2,-526.7994,314.5757
Green R,8,1,-1321.4658,259.78403
Greenfield T,3,1,-796.98395,103.73085
Greensmith L,8,1,-1262.9678,479.47543
Greulich R,8,1,-482.64423,-936.97406
Griffiths M,8,4,-685.3988,164.95117
Grossniklaus M,8,1,1254.0616,-769.61084
Grüter C,6,2,-688.043,-323.8541
Gschwandtner T,8,1,923.6354,-497.95895
Guidotti R,8,1,199.90648,-1138.6099
Guo H,4,1,1462.0875,-81.85923
Gupta A,8,1,-299.62817,-975.98804
Haagen J,6,1,-41.83879,29.10981
Hadzagic M,10,2,380.36832,-1009.24146
Hafezparast M,2,1,-1473.6694,219.28873
Haider J,3,1,-39.813507,340.83
Halama N,8,1,-938.9844,-1120.6246
Hall C,8,1,-586.8505,-780.71747
Hall K,6,1,1079.4479,495.61792
Hama L,3,1,-307.73325,843.9456
Harris S,19,3,-418.61865,295.80563
Hashmani M,2,1,-992.90125,528.17456
Hauser H,4,2,497.07748,611.0827
Hauthal E,5,1,783.6942,-129.22127
Haverkort H,5,1,968.10565,976.6664
Hecker D,12,3,629.2524,-637.7569
Hemery P,3,1,580.51685,1370.6759
Hemsley-Flint F,7,1,510.06586,1149.8066
Henkin R,10,5,241.58705,146.54109
Henry K,5,3,-235.21385,-689.6697
Henry-Riche N,7,1,855.9673,529.6053
Henzi P,4,1,-21.858692,-942.50806
Herbert A,5,1,-1337.2179,606.4182
Herpel E,8,1,-528.0487,-1238.1438
Herranz R,7,1,613.4671,-841.55475
Heurich M,2,1,-104.10861,-1085.1124
Hill M,7,1,-995.72174,-762.67096
Hill S,12,2,-486.16537,84.80472
Hinrichs U,6,1,1151.908,587.53436
Hochegger H,5,1,-1221.87,582.4689
Hoffmann E,1,1,-821.3832,-874.126
Holmes G,10,3,-207.6795,-428.94724
Honess D,8,1,-562.6732,173.34494
Hong S,7,1,-262.26047,-1255.9923
Hu Y,3,1,1354.8495,50.38294
Huang H,6,1,876.38824,-189.76851
Huang Y,5,2,-494.4083,851.0545
Huron S,6,1,1112.7086,679.5006
Hurter C,4,1,688.6055,-746.99884
Hyde J,2,1,177.06358,254.93137
Ibrahim J,5,3,-631.66956,73.485
Ilczyszyn A,6,1,-1390.9938,152.12878
Imiya A,7,2,-1188.8356,-865.66284
Ingham P,8,1,-398.6288,-528.7325
Isenberg P,5,1,494.4148,760.8767
Isenberg T,5,1,425.74365,853.3598
Jaffari R,2,1,-861.0885,345.15433
Janetzko H,18,3,1113.8684,-594.8739
Jankowski P,7,4,643.08154,-295.4229
Janssens D,6,1,81.10148,-675.85004
Jawaid M,8,4,-1040.1049,843.0547
Jena A,7,1,949.57574,50.943817
Jern M,8,1,374.8963,3.9199488
Ji T,9,2,-794.5029,-498.11456
Jia G,7,1,-1063.839,-676.60455
Jiang L,7,1,-278.06573,-1374.5883
Jianu R,41,17,1070.2686,269.1051
Jones M,5,6,-1058.7854,-253.73212
Jones S,11,3,131.16052,999.64545
Jousselme A-L,10,2,207.38689,-890.77374
Jupp T,5,1,681.5983,852.0986
Jäger D,8,1,-891.254,-1031.8253
Kachkaev A,16,12,61.705116,758.2755
Kadirkamanathan V,13,4,-297.9593,-215.68544
Kam T,3,1,63.884605,1266.5208
Kanai J,2,1,1285.5258,745.22034
Kanthou C,23,14,-297.71182,275.02164
Karabağ C,13,13,-1155.2798,-43.852184
Kather J,18,6,-670.19446,-1160.4972
Katner K,1,1,1125.3306,154.71384
Keim D,20,7,862.644,-395.48862
Kelly M,3,3,476.36456,958.40814
Kerren A,5,1,976.6729,-629.5904
Kerzner E,4,1,78.299286,1112.5974
Khalili N,2,1,861.1327,951.12024
Khalili-Shavarini N,4,1,393.5888,1185.9756
Kim E,8,2,-742.34357,1183.8102
Kim J,8,1,333.89383,-1303.3944
Kindermann J,8,1,363.1168,-183.28938
Kisilevich S,5,1,780.86505,-681.7072
Klippel A,4,1,-87.278755,628.06744
Knapp K,9,3,-1295.0281,101.697876
Knauer C,3,1,-425.8731,1430.9893
Kobourov S,2,2,1404.6592,499.8573
Koh L,3,1,167.53157,1332.3635
Komarov A,5,1,672.0579,-1299.0563
Kornfeld A,2,1,-288.66425,1303.0118
Koumparos A,4,1,194.9437,-465.46838
Kraak M,18,4,561.3247,114.83962
Krisam J,8,1,-406.06696,-1240.3468
Kudo M,7,2,-1252.2933,-782.99493
Kureshi I,5,1,968.72546,-982.2856
Kyriacou P,5,3,-1254.582,-368.39633
Kölzsch A,4,1,358.06604,954.9258
Lahtinen K,5,3,30.842657,631.4879
Laidlaw D,4,2,1481.8282,-182.45564
Laird A,8,1,1329.1246,412.00745
Lammarsch T,6,1,696.3507,-488.77527
Lampis M,2,2,-965.4781,1119.8159
Lamprecht A,8,1,1213.7847,-669.0817
Layton J,8,1,-520.74786,-692.5415
Leandrou S,5,3,-1398.6381,-493.68918
Lee K,5,1,1029.8346,-1061.7638
Lefley D,15,3,-333.07715,375.41458
Li J,10,5,226.91312,-344.80194
Liatsis P,4,2,-1158.279,871.249
Liebig T,4,1,835.5812,-1048.8772
Liebman A,7,1,833.9622,430.75406
Liem J,2,1,1125.0199,907.2028
Lieschke G,6,2,-305.82925,-614.19885
Liu F,6,1,-80.61354,-682.26935
Liu S,7,1,-188.5092,-1342.3088
Llado X,7,2,-1305.6593,-695.01544
Lloyd D,2,5,-373.42914,934.46484
Lomax N,3,1,-255.82962,750.5379
Longley P,2,2,328.59604,1435.8701
Lowe R,5,1,738.322,944.5349
Luedde T,8,1,-648.4019,-1277.2144
Lunt S,16,4,-267.92587,63.860126
Löffler M,7,3,157.53737,1222.2245
MacEachren A,6,2,670.8582,-187.13487
Maciejewski R,4,1,-89.74133,-574.31775
Madabhushi A,1,1,-832.73724,-8.8871
Madzvamuse A,4,2,-1027.1526,107.89703
Mamais I,5,1,-1348.6318,-319.06995
Mansmann F,7,1,916.8997,-840.79376
Marcos R,7,1,561.01465,-749.6178
Marcotti S,5,2,-781.8981,952.09296
Marien P,5,1,-945.37085,-5.2566466
Markovic N,4,1,-147.85535,-992.8739
Martinez M,8,1,392.9642,-529.84515
Martinez-Lemus L,16,4,-951.5509,-520.42773
Marx A,10,2,-571.9951,-1143.8352
Mason D,7,2,-1128.6506,-775.0113
Mason J,4,1,-73.945564,457.71948
Mayr E,8,1,60.02816,-156.45934
McCurdy N,2,1,-168.46492,1328.4335
Medyckyj-Scott D,7,1,658.47217,1051.8608
Meier S,7,1,190.5437,579.21246
Merryweather A,2,1,-446.88376,1118.787
Meulemans W,9,4,612.9502,769.14716
Meyer M,5,4,-40.836426,1083.964
Mian A,1,1,-1476.1859,-154.96497
Mikhailov A,5,1,769.7893,-1254.24
Miksch S,20,4,528.8706,-294.58627
Miller N,6,6,-1036.3976,-115.00283
Mills B,2,1,-53.68724,1301.5391
Mitchell C,8,1,-534.0836,743.6676
Mladenov M,4,1,1130.0935,-786.9523
Moat J,2,1,444.32614,1410.8094
Mock M,4,1,1157.7251,-881.1242
Morgades P,2,1,228.23965,1445.4229
Mountain D,4,2,474.8307,1259.3455
Mues V,4,1,-27.564257,-1164.1925
Mukherjee D,6,1,-109.03222,220.58167
Muntoni F,8,1,-1220.0325,305.38217
Nadimi S,4,1,-767.3963,535.09393
Nanni M,12,2,174.38506,-1251.7942
Narejo S,5,1,-1081.9464,951.6326
Navarra C,7,1,996.9903,-759.62616
Nebiker S,2,2,-201.04488,639.3998
Ngan K,8,3,-1226.1177,179.02295
Nguyen P,26,8,326.84265,87.08972
Nolet B,4,1,231.0986,747.85895
Nouri J,3,1,-938.531,625.19574
Nowell C,8,1,-609.6721,-633.76196
O'Donnell J,8,1,-309.84512,-772.1589
Okoe M,10,5,1249.0258,349.19238
Olliverre N,4,1,-1118.6973,533.58777
Olteanu Raimond A,5,1,591.88,-1069.0387
Orabi W,3,1,1286.6863,528.62244
Ortega-Ruiz M,3,3,-1361.9142,-193.34575
Ostermann F,6,1,1029.6705,-197.71826
O’Sullivan C,4,2,-666.8641,491.08716
Parker C,2,1,124.22266,1466.9155
Pase L,19,4,-415.5701,-748.2719
Paterson A,7,1,507.0411,329.87006
Patterson F,3,1,865.67316,-1182.0894
Pattichis C,5,3,-1427.2517,-388.56046
Pazhakh V,8,1,-272.9564,-865.98285
Peddie C,5,5,-1254.776,-231.37321
Pedreschi D,5,1,521.121,-1164.9678
Pelekis N,10,3,204.18922,-651.60583
Pennington K,9,2,-849.44684,-588.217
Perin C,13,3,746.72894,524.27655
Persoone G,3,1,-388.335,656.1279
Petroudi S,5,3,-1301.3722,-464.12582
Pettyjohn K,6,2,-189.48552,-74.5868
Piccolotto N,8,1,973.22217,-403.8124
Pinheiro R,3,1,1250.891,651.6753
Pink H,5,1,-1227.9031,784.5144
Pirzada N,5,1,-939.5497,854.42126
Plumbley M,10,3,-171.15775,1094.1594
Pohl M,11,2,183.02727,30.935902
Pollock K,9,2,-712.74,-568.9585
Pradhananga N,3,1,1358.6984,620.5774
Prise V,8,2,-599.384,-229.85452
Purves R,16,4,835.9663,106.99136
P‚àö‚àÇlitz C,9,2,1030.7759,-530.73584
Radburn R,6,10,-4.4064074,853.6312
Rajani R,5,3,-1106.2764,759.58264
Ramirez Perez F,13,3,-968.2656,-618.9444
Ramos-Lopez C,5,1,-220.23322,-555.18134
Raper J,1,1,42.032616,1387.9556
Ratnieks F,6,2,-837.84686,-240.10945
Raubal M,6,1,1295.7438,-196.1773
Ray C,10,2,400.81125,-906.58453
Ray K,8,1,1396.1265,174.38394
Reece J,4,1,-919.746,-311.98053
Reilly M,8,1,-1307.8226,377.39764
Reimer A,5,2,883.78827,1072.5934
Renshaw S,22,9,-306.5701,-454.6992
Reyes Aldasoro C,216,127,-776.9927,-128.49152
Rhyne T-M,4,1,322.32913,-426.07578
Riaz A,4,1,-1061.7545,636.1982
Riedel M,8,1,1463.3049,262.54144
Rigby M,7,1,660.44025,236.22417
Rind A,6,1,731.7461,-587.04785
Ringrose C,6,1,-1481.1218,-31.71622
Rinzivillo S,9,6,507.92258,-982.73804
Ritsos P,8,1,674.6611,339.1692
Robertson A,7,1,-169.96014,-303.67822
Robinson A,5,1,617.0505,19.312778
Rodenberg J,2,1,-739.18866,306.26007
Rodriguez N,6,1,1243.8533,-103.51125
Rogers K,8,1,-674.80084,-720.73566
Rojas-Moraleda R,8,1,-854.6645,-1189.8086
Rooney C,7,4,312.40222,543.34894
Roskosch P,5,1,1072.1995,-686.2824
Rossor A,8,1,-1184.9824,413.2384
Rote G,3,2,-212.5355,1469.3081
Rusu A,3,1,1469.1094,19.24601
Rzazewski P,8,2,-649.9166,1321.4331
Sacha D,6,1,1301.787,-354.06686
Salmon L,8,1,206.92635,-1017.485
Santipantakis G,11,3,332.27582,-628.30896
Sathiyanarayanan M,2,1,68.90461,298.26733
Savas D,4,2,-718.931,5.6111298
Scarlatti D,8,2,520.945,-883.94293
Schad L,5,1,-603.55096,-1016.31366
Schiewe J,2,1,-371.65216,1209.7871
Schreck T,35,7,1075.1375,-294.28397
Schuck A,4,1,84.13945,-1160.2745
Schulze K,8,1,-394.40643,-1000.33185
Schumann H,12,4,426.4106,-78.36349
Scoto M,8,1,-1418.3442,411.85962
Sekula P,4,1,-121.61355,-795.6642
Shi C,7,1,-221.32161,-1133.3745
Shurkhovetskyy G,3,1,870.6808,-934.67993
Sietinsone L,7,1,546.13153,1049.8539
Sikora F,8,2,-627.80164,1218.6271
Silva N,6,1,1233.3923,-2.6006744
Simoes F,8,1,-1369.4414,499.73477
Skupin A,8,1,83.32369,-248.01328
Slabaugh G,13,13,-994.97003,725.51385
Slingsby A,86,81,350.05118,755.77386
Smith I,5,1,936.3454,-1108.1562
Smith S,8,1,1213.6185,249.19205
Solis-Lemus J,15,9,-717.8094,824.15717
Sowers J,10,2,-1052.6705,-491.3439
Speckmann B,19,6,745.2111,730.92316
Spretke D,7,1,1022.3214,-862.84
Staals F,4,1,584.8796,1264.6438
Stange H,7,4,768.36426,-955.14185
Stasko J,5,1,1020.8934,863.747
Staykova T,5,1,1090.1909,-966.7621
Steele A,8,1,-372.45648,481.6545
Stein M,8,1,1316.2966,-666.34094
Steinbrecher M,6,1,1201.1208,-327.43002
Stephens D,8,1,285.65616,314.06857
Stephenson D,5,1,794.10095,842.44073
Stiff T,5,1,-1171.611,672.12805
Stolper C,5,1,1108.8376,787.9599
Strachan J,4,2,257.84784,1009.04846
Stramer B,6,7,-828.9214,844.4901
Streit M,8,1,955.12787,-298.18478
Styles V,4,2,-912.6453,115.2161
Suarez Carmona M,8,1,-796.49646,-1109.0187
Sun Y,5,1,22.173534,-342.9994
Suschnigg J,8,1,757.8802,-385.31522
Sutherland M,8,1,1435.3916,372.74466
Symanzik J,5,1,705.3344,-1050.5724
Sánchez-Sánchez B,5,2,-685.6093,943.63074
Sîrbu A,8,1,273.59015,-1221.3757
Taggart D,3,1,1488.7432,121.79644
Talton O,8,1,-822.58936,-761.169
Tampakis P,5,1,140.40271,-566.12555
Tao Y,6,1,1006.4463,380.18765
Tate N,2,2,-9.87457,1477.4509
Ter-Sarkisov A,5,2,-1121.1656,196.40517
Theodoridis Y,20,5,194.06491,-759.0311
Thomasse S,8,1,-860.5963,1210.0505
Thonnard O,11,5,277.7871,-133.21309
Tidhar D,10,2,-254.16592,996.5373
Tominski C,9,2,165.67627,-123.33758
Tozer G,36,27,-366.49677,-1.7319919
Turkay C,39,23,381.5679,186.04005
Urwin T,7,1,618.8769,1160.8616
Valous N,8,1,-756.9655,-1231.7982
Van Goethem A,8,4,721.7013,1142.7255
Vander Laan Z,4,1,-119.26894,-895.6955
Varma S,8,1,-597.02673,-889.54736
Vasilakos I,3,1,-882.308,744.10034
Venkataraman C,4,2,-907.4239,233.63028
Verbeek K,6,1,967.5241,476.14883
Verhoeven J,6,6,-1052.2274,-5.040323
Vidale P,4,2,299.96112,858.20154
Vigilante A,8,1,-512.20245,644.1647
Vives S,2,1,-468.72668,1013.08356
Vlachou A,11,3,294.0244,-735.0773
Vojnovic B,8,3,-495.85226,-202.41962
Von Landesberger T,14,5,754.3149,-254.68362
Vouros G,15,5,400.24783,-712.504
Vrotsou K,7,2,890.10864,-731.9235
Vuillemot R,5,1,1002.3002,742.637
Wachowicz M,4,1,94.7143,-29.07898
Walker R,8,1,235.03786,461.9107
Walmsley S,8,1,-509.2505,-558.3335
Wang X,8,1,-593.8055,-494.5828
Wang Y,12,2,3.0777504,-733.9293
Warburton F,8,1,-636.46796,709.6317
Weber H,10,3,561.76556,-178.11354
Weibel R,4,2,689.0052,97.0422
Weis C,10,3,-456.23254,-1149.1462
Weiskopf D,9,2,1150.319,-214.51358
Weston A,5,5,-1145.3777,-162.24553
Wets G,6,1,5.0130453,-828.5812
Weyde T,13,4,-1.517939,994.51184
Whyte M,12,2,-475.7971,-442.0182
Williams L,11,7,-201.0459,263.99957
Wils L,2,1,-650.54913,381.73196
Wilson I,8,3,-567.62805,-117.69243
Wise S,3,1,1000.9931,1099.6008
Wittmann C,8,1,-702.3394,-823.56714
Wlodkovic D,3,1,-582.0265,935.5437
Wlodkowic D,3,1,-417.25684,754.9221
Wolff D,10,3,-143.08415,1000.499
Wong W,12,5,322.46216,415.40424
Wood J,91,88,593.9944,665.0572
Wright V,7,1,-291.194,-68.27078
Wrobel S,20,7,539.1915,-421.85056
Wu H-H,9,2,-774.1294,-662.5948
Wu Y,7,1,-98.19279,-1289.1101
Xu K,8,1,133.7756,483.8073
Yan Y,3,1,-832.9906,639.083
Yang G,4,1,-1065.7112,439.412
Yang J,7,1,1080.3746,32.681435
Yin T,2,1,-991.5581,334.90195
Youssef A,1,1,-350.8054,-1109.8622
Yuan X,7,2,289.4876,-261.82422
Zhang K,5,2,71.42561,-463.4337
Zhang Q,6,1,433.49402,416.88306
Zhang X,7,1,-202.1256,-196.57382
Zhang Y,7,2,-1342.8472,-600.4202
Zhao Y,10,2,-87.69416,-367.32672
Zheng Y,8,1,180.91866,371.0882
Ziemlicki C,5,1,633.36847,-966.4642
Zimmermann P,8,1,1362.3741,-462.17963
Zollino M,5,1,-1113.4558,-961.5296
Zouaoui J,5,1,551.548,-80.93475
Zöllner F,5,2,-503.07776,-1050.6049
de Chaumont F,8,1,-700.91235,626.649
van Eeden F J,8,1,-577.98425,-378.8221
van Kreveld M,6,1,919.43475,334.68152
van Loon E,2,5,40.30836,415.57385"""
        |> fromCSV
```

### Appendix E: Sankey Diagrams with elm-vega

_See source code of this document for details._

```elm {l=hidden}
-- Record for storing Sankey customisation options.


type alias Sankey =
    { data : V.DataTable
    , dataSource : String
    , oName : String
    , dName : String
    , vName : String
    , width : Float
    , height : Float
    , oLabel : String
    , dLabel : String
    , cScale : List V.ScaleProperty
    , signal : List V.Spec -> List V.Spec
    , trans : List V.Transform
    }



-- The main Sankey diagram builder


defaultSankey =
    { data =
        (V.dataFromColumns "sankeyData" []
            << V.dataColumn "o" (V.vStrs [ "o" ])
            << V.dataColumn "d" (V.vStrs [ "d" ])
            << V.dataColumn "v" (V.vNums [ 1 ])
        )
            []
    , dataSource = "sankeyData"
    , oName = "o"
    , dName = "d"
    , vName = "v"
    , width = 600
    , height = 600
    , oLabel = "state 1"
    , dLabel = "state 2"
    , cScale = [ V.scType V.scOrdinal, V.scRange V.raCategory ]
    , signal = identity
    , trans = []
    }


sankeyBuilder : Sankey -> Spec
sankeyBuilder sankey =
    let
        ds =
            V.dataSource <|
                (sankey.data
                    |> V.transform
                        (sankey.trans
                            ++ [ V.trFormula ("datum['" ++ sankey.oName ++ "']") "stk1"
                               , V.trFormula ("datum['" ++ sankey.dName ++ "']") "stk2"
                               , V.trFormula ("datum['" ++ sankey.vName ++ "']") "size"
                               ]
                        )
                )
                    :: [ V.data "nodes" [ V.daSource sankey.dataSource ]
                            |> V.transform
                                [ V.trFilter (V.expr "!groupSelector || groupSelector.stk1 == datum.stk1 || groupSelector.stk2 == datum.stk2")
                                , V.trFormula "datum.stk1+datum.stk2" "key"
                                , V.trFoldAs [ V.field "stk1", V.field "stk2" ] "stack" "grpId"
                                , V.trFormula "datum.stack == 'stk1' ? datum.stk1+' '+datum.stk2 : datum.stk2+' '+datum.stk1" "sortField"
                                , V.trStack
                                    [ V.stField (V.field "size")
                                    , V.stSort [ ( V.field "sortField", V.descend ) ]
                                    , V.stGroupBy [ V.field "stack" ]
                                    ]
                                , V.trFormula "(datum.y0+datum.y1)/2" "yc"
                                ]
                       , V.data "groups" [ V.daSource "nodes" ]
                            |> V.transform
                                [ V.trAggregate
                                    [ V.agFields [ V.field "size" ]
                                    , V.agOps [ V.opSum ]
                                    , V.agGroupBy [ V.field "stack", V.field "grpId" ]
                                    , V.agAs [ "total" ]
                                    ]
                                , V.trStack
                                    [ V.stField (V.field "total")
                                    , V.stSort [ ( V.field "grpId", V.descend ) ]
                                    , V.stGroupBy [ V.field "stack" ]
                                    ]
                                , V.trFormula "scale('yScale', datum.y0)" "scaledY0"
                                , V.trFormula "scale('yScale', datum.y1)" "scaledY1"
                                , V.trFormula "datum.stack == 'stk1'" "rightLabel"
                                , V.trFormula "datum.total/domain('yScale')[1]" "percentage"
                                ]
                       , V.data "destinationNodes" [ V.daSource "nodes" ]
                            |> V.transform [ V.trFilter (V.expr "datum.stack == 'stk2'") ]
                       , V.data "edges" [ V.daSource "nodes" ]
                            |> V.transform
                                [ V.trFilter (V.expr "datum.stack == 'stk1'")
                                , V.trLookup "destinationNodes" (V.field "key") [ V.field "key" ] [ V.luAs [ "target" ] ]
                                , V.trLinkPath
                                    [ V.lpOrient V.orHorizontal
                                    , V.lpShape V.lsDiagonal
                                    , V.lpSourceX (V.fExpr "scale('xScale', 'stk1') + bandwidth('xScale')")
                                    , V.lpSourceY (V.fExpr "scale('yScale', datum.yc)")
                                    , V.lpTargetX (V.fExpr "scale('xScale', 'stk2')")
                                    , V.lpTargetY (V.fExpr "scale('yScale', datum.target.yc)")
                                    ]
                                , V.trFormula "range('yScale')[0]-scale('yScale', datum.size)" "strokeWidth"
                                , V.trFormula "datum.size/domain('yScale')[1]" "percentage"
                                ]
                       ]

        si =
            V.signals
                << V.signal "groupHover"
                    [ V.siValue (V.vStr "{}")
                    , V.siOn
                        [ V.evHandler [ V.esSelector (V.str "@groupMark:mouseover") ]
                            [ V.evUpdate "{stk1:datum.stack=='stk1' && datum.grpId, stk2:datum.stack=='stk2' && datum.grpId}" ]
                        , V.evHandler [ V.esSelector (V.str "mouseout") ] [ V.evUpdate "{}" ]
                        ]
                    ]
                << V.signal "groupSelector"
                    [ V.siValue V.vFalse
                    , V.siOn
                        [ V.evHandler [ V.esSelector (V.str "@groupMark:click!") ]
                            [ V.evUpdate "{stack:datum.stack, stk1:datum.stack=='stk1' && datum.grpId, stk2:datum.stack=='stk2' && datum.grpId}" ]
                        , V.evHandler
                            [ V.esObject [ V.esType V.etClick, V.esMarkName "groupReset" ]
                            , V.esObject [ V.esType V.etDblClick ]
                            ]
                            [ V.evUpdate "false" ]
                        ]
                    ]
                << sankey.signal

        sc =
            V.scales
                << V.scale "xScale"
                    [ V.scType V.scBand
                    , V.scDomain (V.doStrs (V.strs [ "stk1", "stk2" ]))
                    , V.scRange V.raWidth
                    , V.scPaddingInner (V.num 0.95)
                    , V.scPaddingOuter (V.num 0.05)
                    ]
                << V.scale "yScale"
                    [ V.scType V.scLinear
                    , V.scDomain (V.doData [ V.daDataset "nodes", V.daField (V.field "y1") ])
                    , V.scRange V.raHeight
                    ]
                << V.scale "cScale" sankey.cScale
                << V.scale "stackNames"
                    [ V.scType V.scOrdinal
                    , V.scDomain (V.doStrs (V.strs [ "stk1", "stk2" ]))
                    , V.scRange (V.raStrs [ sankey.oLabel, sankey.dLabel ])
                    ]

        ax =
            V.axes
                << V.axis "xScale"
                    V.siTop
                    [ V.axDomain V.false
                    , V.axTicks V.false
                    , V.axLabelPadding (V.num 10)
                    , V.axEncode
                        [ ( V.aeLabels
                          , [ V.enUpdate [ V.maText [ V.vScale "stackNames", V.vField (V.field "value") ] ] ]
                          )
                        ]
                    ]
                << V.axis "yScale"
                    V.siLeft
                    [ V.axDomain V.false
                    , V.axTicks V.false
                    , V.axLabels V.false
                    ]

        mk =
            V.marks
                -- Left and right stacks representing each transition combination between the two states.
                << V.mark V.rect
                    [ V.mFrom [ V.srData (V.str "nodes") ]
                    , V.mEncode
                        [ V.enEnter
                            [ V.maWidth [ V.vScale "xScale", V.vBand (V.num 1) ]
                            , V.maX [ V.vScale "xScale", V.vField (V.field "stack") ]
                            , V.maY [ V.vScale "yScale", V.vField (V.field "y0") ]
                            , V.maY2 [ V.vScale "yScale", V.vField (V.field "y1") ]
                            , V.maFill [ V.vScale "cScale", V.vField (V.field "grpId") ]
                            , V.maStroke [ V.black ]
                            , V.maStrokeWidth [ V.vNum 0.2 ]
                            ]
                        ]
                    ]
                -- Left and right stack groups (group of nodes with common origin or destination).
                << V.mark V.rect
                    [ V.mName "groupMark"
                    , V.mFrom [ V.srData (V.str "groups") ]
                    , V.mEncode
                        [ V.enEnter
                            [ V.maFillOpacity [ V.vNum 0 ]
                            , V.maFill [ V.vStr "white" ]
                            , V.maWidth [ V.vScale "xScale", V.vBand (V.num 1) ]
                            , V.maStroke [ V.vStr "white" ]
                            , V.maStrokeWidth [ V.vNum 2 ]
                            , V.maCornerRadius [ V.vNum 2 ]
                            ]
                        , V.enUpdate
                            [ V.maX [ V.vScale "xScale", V.vField (V.field "stack") ]
                            , V.maY [ V.vField (V.field "scaledY0") ]
                            , V.maY2 [ V.vField (V.field "scaledY1") ]
                            , V.maTooltip [ V.vSignal "datum.grpId + '   ' + format(datum.total, ',.0f') + '   (' + format(datum.percentage, '.1%') + ')'" ]
                            ]
                        , V.enHover
                            [ V.maStrokeOpacity [ V.vNum 1 ]
                            ]
                        ]
                    ]
                -- The Sankey flow lines.
                << V.mark V.path
                    [ V.mName "edgeMark"
                    , V.mFrom [ V.srData (V.str "edges") ]
                    , V.mClip (V.clEnabled V.true)
                    , V.mEncode
                        [ V.enUpdate
                            [ V.maStroke
                                [ V.ifElse "groupSelector && groupSelector.stack=='stk1'"
                                    [ V.vScale "cScale", V.vField (V.field "stk2") ]
                                    [ V.vScale "cScale", V.vField (V.field "grpId") ]
                                ]
                            , V.maStrokeWidth [ V.vField (V.field "strokeWidth") ]
                            , V.maPath [ V.vField (V.field "path") ]
                            , V.maStrokeOpacity [ V.vSignal "!groupSelector && (groupHover.stk1 == datum.stk1 || groupHover.stk2 == datum.stk2) ? 0.9 : 0.3" ]
                            , V.maZIndex [ V.vSignal "!groupSelector && (groupHover.stk1 == datum.stk1 || groupHover.stk2 == datum.stk2) ? 1 : 0" ]
                            , V.maTooltip [ V.vSignal "datum.stk1 + ' → ' + datum.stk2 + '    ' + format(datum.size, ',.0f') + '   (' + format(datum.percentage, '.1%') + ')'" ]
                            ]
                        , V.enHover [ V.maStrokeOpacity [ V.vNum 1 ] ]
                        ]
                    ]
                -- Group labels.
                << V.mark V.text
                    [ V.mFrom [ V.srData (V.str "groups") ]
                    , V.mInteractive V.false
                    , V.mEncode
                        [ V.enUpdate
                            [ V.maX [ V.vSignal "scale('xScale', datum.stack) + (datum.rightLabel ? bandwidth('xScale') + 8 : -8)" ]
                            , V.maY [ V.vSignal "(datum.scaledY0 + datum.scaledY1)/2" ]
                            , V.maAlign [ V.vSignal "datum.rightLabel ? 'left' : 'right'" ]
                            , V.maBaseline [ V.vMiddle ]
                            , V.maFontWeight [ V.vStr "bold" ]
                            , V.maText [ V.vSignal "abs(datum.scaledY0-datum.scaledY1) > 13 ? datum.grpId : ''" ]
                            ]
                        ]
                    ]
    in
    V.toVega
        [ V.width sankey.width, V.height sankey.height, ds, si [], sc [], ax [], mk [] ]


categoricalDomainMap : List ( String, String ) -> List V.ScaleProperty
categoricalDomainMap scaleDomainPairs =
    -- Convenience function for creating a custom categorical colour scheme
    -- from a list of category-colour tuples.
    let
        ( domain, range ) =
            List.unzip scaleDomainPairs
    in
    [ V.scType V.scOrdinal
    , V.scDomain (V.doStrs (V.strs domain))
    , V.scRange (V.raStrs range)
    ]
```

### Appendix F: London Borough Locations and IDs

_See source code of this document for details._

```elm {l=hidden}
locationTable : Table
locationTable =
    """GSS,Name,ShortName,LACode,row,col,longitude,latitude
E09000001,City of London,Cty,00AA,4,5,-0.092,51.514
E09000002,Barking and Dagenham,Bar,00AB,4,8,0.133,51.544
E09000003,Barnet,Brn,00AC,2,4,-0.210,51.616
E09000004,Bexley,Bxl,00AD,5,8,0.142,51.461
E09000005,Brent,Brt,00AE,3,3,-0.268,51.559
E09000006,Bromley,Brm,00AF,6,6,0.052,51.372
E09000007,Camden,Cmd,00AG,3,4,-0.157,51.546
E09000008,Croydon,Crd,00AH,6,5,-0.087,51.355
E09000009,Ealing,Elg,00AJ,3,2,-0.331,51.522
E09000010,Enfield,Enf,00AK,1,5,-0.087,51.651
E09000011,Greenwich,Grn,00AL,5,7,0.056,51.474
E09000012,Hackney,Hck,00AM,3,6,-0.063,51.552
E09000013,Hammersmith and Fulham,Hms,00AN,4,2,-0.221,51.495
E09000014,Haringey,Hgy,00AP,2,5,-0.107,51.590
E09000015,Harrow,Hrw,00AQ,2,3,-0.341,51.598
E09000016,Havering,Hvg,00AR,3,8,0.220,51.563
E09000017,Hillingdon,Hdn,00AS,3,1,-0.446,51.542
E09000018,Hounslow,Hns,00AT,4,1,-0.366,51.468
E09000019,Islington,Isl,00AU,3,5,-0.110,51.548
E09000020,Kensington and Chelsea,Kns,00AW,4,3,-0.192,51.501
E09000021,Kingston upon Thames,Kng,00AX,6,3,-0.287,51.388
E09000022,Lambeth,Lam,00AY,5,4,-0.118,51.454
E09000023,Lewisham,Lsh,00AZ,5,6,-0.020,51.448
E09000024,Merton,Mrt,00BA,6,4,-0.197,51.410
E09000025,Newham,Nwm,00BB,4,7,0.038,51.527
E09000026,Redbridge,Rdb,00BC,3,7,0.076,51.586
E09000027,Richmond upon Thames,Rch,00BD,5,2,-0.312,51.443
E09000028,Southwark,Swr,00BE,5,5,-0.074,51.474
E09000029,Sutton,Stn,00BF,7,4,-0.178,51.362
E09000030,Tower Hamlets,Tow,00BG,4,6,-0.035,51.516
E09000031,Waltham Forest,Wth,00BH,2,6,-0.013,51.594
E09000032,Wandsworth,Wns,00BJ,5,3,-0.186,51.452
E09000033,Westminster,Wst,00BK,4,4,-0.160,51.513"""
        |> fromCSV
```

### Appendix G: London Commuter Flow Data

_See source code of this document for details._

```elm {l=hidden}
odMatrix : Table
odMatrix =
    """Origin Code,00AA,00AB,00AC,00AD,00AE,00AF,00AG,00AH,00AJ,00AK,00AL,00AM,00AN,00AP,00AQ,00AR,00AS,00AT,00AU,00AW,00AX,00AY,00AZ,00BA,00BB,00BC,00BD,00BE,00BF,00BG,00BH,00BJ,00BK
00AA,1627,6,14,0,16,0,335,3,9,6,3,66,61,16,7,9,10,9,339,62,0,42,15,6,16,0,6,103,3,266,16,16,602
00AB,3641,20434,194,96,178,66,1500,104,188,410,209,980,255,301,55,6415,97,91,1385,439,18,449,132,67,4460,5940,28,926,38,4069,1047,165,3280
00AC,7709,96,44045,34,5467,76,12080,148,1573,4098,101,1576,1431,3842,2623,131,1023,611,5775,1841,68,1129,124,146,387,305,229,1792,44,2511,555,536,16330
00AD,6580,362,132,33681,144,4998,2470,710,188,123,9498,698,333,110,29,394,109,161,1557,603,77,1808,2989,190,574,111,90,4666,170,2825,222,618,7692
00AE,4145,40,6124,28,32105,66,8105,187,6703,426,124,809,3709,730,5102,51,2127,1404,2535,3926,116,1107,147,203,279,116,396,1565,56,1655,210,669,16418
00AF,9855,134,162,3199,201,51289,3780,6268,293,84,2586,728,859,134,59,119,248,285,2094,1241,227,3570,5360,647,419,100,191,6057,796,3385,196,1371,12802
00AG,8795,36,1496,32,1350,60,26535,147,643,295,140,1201,1870,666,330,43,473,473,4987,2267,89,1338,140,139,290,84,195,1784,54,2614,204,588,18829
00AH,5925,85,204,300,329,5152,3248,65033,405,120,523,599,1284,177,97,82,457,581,1752,1478,827,6844,1563,3521,202,64,480,4513,6744,2000,130,4270,10583
00AJ,4855,30,967,33,5263,91,4547,182,42004,223,136,522,8579,267,2601,48,13055,9869,1795,4111,364,1031,174,361,229,79,1578,1328,65,1420,127,944,13967
00AK,5212,217,5642,52,1038,76,5588,136,626,44370,126,2542,850,10199,328,233,457,339,5317,1091,38,806,114,114,661,538,98,1253,47,2141,1710,314,9052
00AL,5614,333,181,5046,252,3196,3038,692,248,192,27753,856,607,178,45,147,154,233,1780,929,58,2292,5571,237,981,126,113,5145,134,3643,244,940,8847
00AM,5181,222,687,56,514,90,7137,128,450,877,241,18706,1165,2335,92,124,203,283,7740,1261,61,1269,240,121,1069,402,149,2023,58,5247,1101,567,9983
00AN,7967,25,259,19,812,56,4009,202,2418,82,63,586,19307,137,163,33,1090,2097,1795,7072,271,1153,122,466,123,33,854,1374,113,1947,66,1908,15305
00AP,5523,144,3016,32,1074,64,10506,213,668,4249,154,3130,1486,20233,268,99,330,450,7948,1889,67,1236,178,136,520,278,180,1857,48,2350,1004,507,13479
00AQ,3162,55,5008,26,9473,47,3675,103,4542,325,62,430,1487,331,27684,29,6169,1141,1395,1160,107,501,71,119,99,44,246,854,34,1013,95,300,7882
00AR,8348,7278,194,325,156,157,2001,131,165,546,260,1302,260,335,69,39760,116,83,1932,358,33,552,137,85,3410,4844,45,1389,50,5030,1173,158,4699
00AS,1661,31,692,18,2672,55,1757,116,8185,153,49,205,1799,110,4688,21,55245,5293,664,925,150,401,79,167,83,35,548,511,58,565,53,248,5062
00AT,2342,19,253,27,807,69,1753,215,5488,92,106,250,3587,87,396,28,12803,34554,752,1800,1006,692,128,303,100,31,7025,657,101,753,41,765,5927
00AU,7931,110,1001,46,538,46,10188,157,479,619,138,2758,1473,1869,111,55,265,334,19946,1672,50,1191,152,131,348,117,118,1883,56,2837,393,490,12835
00AW,10098,22,232,20,548,64,3554,165,861,48,54,445,3679,123,106,18,576,788,1816,15623,112,1029,78,208,68,30,300,1168,67,2957,68,1085,15947
00AX,2875,9,104,35,184,72,1547,638,469,42,58,176,1327,32,68,24,1070,1484,710,847,26649,1028,70,3041,71,31,3788,1038,1190,825,20,2395,5419
00AY,9697,64,406,136,477,826,8090,3143,667,150,595,1469,3376,373,89,49,466,898,4149,4368,527,26512,1344,1830,380,123,855,8267,803,3414,185,8028,24349
00AZ,7251,164,271,1170,332,6677,4963,2457,457,133,4555,1231,1114,277,83,93,238,365,2669,1791,181,5685,26819,583,657,141,259,11118,406,3949,236,1901,13727
00BA,6051,34,181,56,251,316,2952,3193,554,49,176,454,2310,143,68,20,532,848,1411,2223,3516,3383,250,23205,157,31,1334,2282,3743,1704,81,8410,10148
00BB,5009,2702,401,147,513,207,3855,190,431,584,735,2397,741,675,57,934,207,283,2709,1497,86,1154,386,139,24269,2969,126,2089,56,9120,2161,534,8565
00BC,8205,4664,567,141,386,132,3790,173,343,1187,330,2338,581,906,110,3004,212,231,3104,746,46,853,188,100,6031,28666,88,1760,48,6916,5441,262,8122
00BD,4831,15,173,12,342,64,2504,323,1469,41,64,276,3181,53,167,20,3365,6873,1002,1745,3549,1216,84,840,65,12,24074,1213,260,1197,46,1991,8336
00BE,9041,108,319,281,428,933,6003,1213,536,135,1138,1401,1682,271,123,74,312,466,3203,2552,273,8500,3163,566,580,161,313,28224,261,4243,251,2545,16500
00BF,2560,33,81,68,138,514,1450,7602,309,45,101,230,851,51,50,23,385,541,681,882,3122,1827,270,6725,95,26,750,1404,30814,872,41,3217,4691
00BG,10104,370,257,140,192,192,4080,183,291,224,275,2668,935,326,56,304,146,245,3235,1249,58,1090,316,124,1845,407,151,2182,55,23242,562,405,9304
00BH,6337,846,715,79,491,149,5554,253,440,2661,337,3957,832,2467,112,594,206,271,4310,1378,50,1059,228,122,3257,3736,101,1719,41,4789,28062,409,10314
00BJ,13935,34,297,69,422,324,7140,1496,970,116,256,1044,6232,225,129,31,853,1690,3404,6208,1806,5951,403,4329,208,59,2338,4092,992,4024,135,29768,24409
00BK,9521,25,514,17,1229,99,6786,216,925,121,93,745,2369,228,142,26,597,572,2442,4767,90,1381,148,178,170,60,258,1663,61,2851,121,873,36348"""
        |> fromCSV
```
