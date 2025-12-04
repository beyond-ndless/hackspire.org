---
title: Differences between the material versions
permalink: /Differences_between_the_material_versions/
parent: TNS File Format
layout: page
---

I'll use this as a sandbox:

``` xml
<!-- Vous trouverez ici un aperçu du code des différents widgets. Le code est détaillé dans leur page dédiée. -->

<!-- Widget -->
<!-- 8 Types: Calcul/Graphique/Tableur/Stats/Editeur Math/Editeur Basic/Lua/Vernier -->
<!-- De forme générale: -->

<wdgt xmlns:{xy(z)}="urn:TI.{NOM]" type="TI.{NOM}" ver="{VERSION}"> <!-- xyz: Lettres en fonction du widget -->
</wdgt>
<wdgt xmlns:{xyz}="urn:TI.{NOM]" type="TI.{NOM}" ver="{VERSION}"> <!-- xyz: Lettres en fonction du widget -->
</wdgt>
 <wdgt xmlns:xyz="urn:TI.{NOM]" type="TI.{NOM}" ver="{VERSION}"> <!-- xyz: Lettres en fonction du widget -->
</wdgt>

<!-- Calcul -->
<wdgt xmlns:sp="urn:TI.Scratchpad" type="TI.Scratchpad" ver="1.0">
<sp:mFlags>1024</sp:mFlags
><sp:value>0</sp:value>
<sp:auth>ARBRE</sp:auth>
</wdgt>

<!-- Décorticage -->

<wdgt xmlns:sp="urn:TI.Scratchpad" type="TI.Scratchpad" ver="1.0">  <!-- Balises du type <sp:{NOM} -->
<sp:mFlags>1024</sp:mFlags>                                         <!--  -->
<sp:value>0</sp:value>                                              <!--  -->
<sp:auth>ARBRE</sp:auth>                                            <!-- Les lignes de calcul sont stockées là dedans -->
</wdgt>                                                             <!--  -->



<!-- Application Graphiques et géométrie -->
<!-- Condensé de code quand on crée une page sans la modifier (influencé tout de même par les réglages de la nspire): -->
<!-- Balises de la forme <gg:{BLA}> -->

<wdgt xmlns:gg="urn:TI.GeoGrapher" type="TI.GeoGrapher" ver="1.0">

<gg:mFlags>3072</gg:mFlags>
<gg:value>1</gg:value>
<gg:cry>0</gg:cry>
<gg:legal>none</gg:legal>
<gg:schk>false</gg:schk>
<gg:guid>7815A8B6432A45B9ACF11EF2384A352D</gg:guid>
<gg:anim_x>10</gg:anim_x>
<gg:anim_y>35</gg:anim_y>
<gg:chevr>0</gg:chevr>

<gg:fig>

    <gg:ver>10</gg:ver>
    <gg:objs>
        {BALISES DE CONFIG}
    </gg:objs>
    <gg:tool>
        {BALISES DE CONFIG}
    </gg:tool>
    {BALISES DE CONFIG}
</gg:fig>

{BALISES DU TYPE:}
<gg:wdgt type="{TEXTE}"></gg:wdgt>

</wdgt>
</wdgt>
```