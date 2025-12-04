---
title: Document.xml
permalink: /Document.xml/
parent: TNS File Format
layout: page
---

Document.xml contains general infos about the TNS file

``` xml
<?xml version="1.0" encoding="UTF-8" ?>         <!--  XML beginning -->
<doc ver="1.0">                     <!--  -->
    <settings>                  <!-- Most of the document setting  (those you can find in "setting" on your nspire) -->
        <assessmentMode>0</assessmentMode>  <!--  -->
        <lastview>1</lastview>          <!--  -->
        <readOnlyMode>{0,1}</readOnlyMode>  <!-- Activate Read Only Mode (1 if activate)  -->
        <lang>fr</lang>             <!-- Language in which the document was saved -->
        <dfmt>0</dfmt>              <!--  -->
        <tfmt>0</tfmt>              <!--  -->
        <curr>0</curr>              <!--  -->
        <expf>1</expf>              <!--  -->
        <ddig>7</ddig>              <!--  -->
        <angf>1</angf>              <!--  -->
        <exapp>1</exapp>            <!--  -->
        <cplxf>4</cplxf>            <!--  -->
        <unit>1</unit>              <!--  -->
        <vectf>1</vectf>            <!--  -->
        <base>1</base>              <!-- Base: dec/hex/bin -->
        <gg_ddig>4</gg_ddig>            <!-- "gg:" Graph&Geometry settings -->
        <gg_graphang>1</gg_graphang>        <!--  -->
        <gg_geomang>2</gg_geomang>      <!--  -->
        <gg_axisendv>1</gg_axisendv>        <!--  -->
        <gg_tooltipfunman>0</gg_tooltipfunman>  <!--  -->
        <gg_autopoi>1</gg_autopoi>      <!--  -->
        <gg_calcmenu>0</gg_calcmenu>        <!--  -->
        <gg_hideplotlabels>0</gg_hideplotlabels><!--  -->
        <boldnessFactor>0</boldnessFactor>  <!--  -->
    </settings>                 <!--  -->
    <rights></rights>               <!-- Document rights. Can be define in TINCS -->
    <nps>1</nps>                    <!-- Number of problems -->
    <imgsize>0</imgsize>                <!--  -->
</doc>                          <!--  -->
```