---
title: ProblemX.xml
permalink: /ProblemX.xml/
parent: TNS File Format
layout: page
---

# Problem

## Raw

``` xml

<!-- In file ProblemX.xml (X being the problem's number). -->

<prob ns="urn:TI.Problem" ver="1.0" pbname="{ProblemName}">
<sym></sym>
<INSERT-CARDS-HERE…>
</prob>
```

## Explanation

``` xml
<prob               ><!-- I cut the <prob ... > in several lines -->
    ns="urn:TI.Problem" <!--  -->
    ver="1.0"       <!--  -->
    pbname="NomDuProbleme"  <!-- Problem name -->
>               <!--  -->
<sym></sym>             <!-- Vars are stored here -->
</prob>
```

# Card

## Raw

``` xml
<card clay="{0-7}" h1="{0-10000}" h2="{0-10000}" w1="{0-10000}" w2="{0-10000}">
<isDummyCard>0</isDummyCard>
<flag>0</flag>
<INSERT-WIDGETS-HERE…>
</card>
```

## Explanation

``` xml

<card               ><!--  -->
    clay="{0-7}"        <!-- Page layout from 0 (1 widget) to 7 (4 widgets)-->
    h1="{0-10000}"      <!-- Height of the left top widget -->
    h2="{0-10000}"      <!--  -->
    w1="{0-10000}"      <!-- Width of the left top widget -->
    w2="{0-10000}"      <!--  -->
>               <!--  -->
<isDummyCard>0</isDummyCard>    <!--  -->
<flag>0</flag>          <!--  -->
</card>             <!--  -->
```