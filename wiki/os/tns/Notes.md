---
title: Notes
permalink: /Notes/
parent: TNS File Format
layout: page
---

# Notes Meta-data

## Raw

``` xml
<wdgt xmlns:np="urn:TI.Notepad" type="TI.Notepad" ver="2.0">
<np:mFlags>1024</np:mFlags>
<np:value>{TEMPLATE}</np:value>
<np:fmtxt>{TREE}</np:fmtxt>
</wdgt>
```

## Explanation

``` xml
<wdgt xmlns:np="urn:TI.Notepad" type="TI.Notepad" ver="2.0">    <!--  -->
<np:mFlags>1024</np:mFlags>                 <!--  -->
<np:value>{TEMPLATE}</np:value>                 <!-- 1:Q&A 2:Proof 3:Default -->
<np:fmtxt>{TREE}</np:fmtxt>                 <!-- Text is in this tree -->
</wdgt>                             <!--  -->
```

# Notes Text-tree

Text is stored in a tree beginning by <r2dtotree> and ending by
</r2dtotree>.

All \< , \> , " , ect... should be written as \< , \>... Here they are
transformed so you can read easily.

## Doc

``` xml
<node name="1doc">{paragraphs}</node> <!-- If you put several 1doc, only the last one will be displayed (at least in "default" mode) -->
```

## Paragraph

``` xml
<node name="1para">{lines}</node> <!-- New paragraph each time you're forcing a new line (pressing "enter") -->
```

## Line

``` xml
<node name="1rtline">{words}</node> <!-- Stupidly recomputed by the nspire, so its often useless to create a new line -->
```

## Word

Several types of words: 1word : normal // 1keyword : bold // 1subhead :
italic // 1title : underline // 1subscrp : subscript // 1supersc :
superscript . Thus, you can't have a bold underline word.

``` xml
<leaf name="1{typeofword}">{text}</leaf> <!-- There should be a leaf per word, but you can put several word in a leaf (using regular spaces) -->
```

## Cursor

Put this anywhere in the text (see raw examples). Putting more than one
crashes the nspire.

``` xml
<cursor index="0"/>
```

# Raw exemple

``` xml
<wdgt xmlns:np="urn:TI.Notepad" type="TI.Notepad" ver="2.0">
    <np:mFlags>1024</np:mFlags>
    <np:value>3</np:value>
    <np:fmtxt>
        <r2dtotree>
            <node name="1doc">
                <node name="1para">
                    <node name="1rtline">
                        <leaf name="1word">
                            L1
                        </leaf>
                    </node>
                </node>

                <node name="1para">
                    <node name="1rtline">
                        <leaf name="1word">
                            L2
                        </leaf>
                    </node>
                </node>

                <node name="1para">
                    <node name="1rtline">
                        <leaf name="1word">
                            L3
                        </leaf>
                    </node>
                </node>

                <node name="1para">
                    <node name="1rtline">
                        <leaf name="1word" continued="1">
                            llllllllllllllllllllllllllllllllllllllllllllllllllllllllllllllllllllllllllllllllllllllllllllllllll
                        </leaf>
                    </node>
                    <node name="1rtline">
                        <leaf name="1word" continuation="1">
                            llllllllllllllllllllllllllllllllllllllllllllllllllllllll
                        </leaf>
                    </node>
                </node>

                <node name="1para">
                    <node name="1rtline">
                        <leaf name="1word"/>
                    </node>
                </node>

                <node name="1para">
                    <node name="1rtline">
                        <leaf name="1word">
                            L10!
                        <cursor index="0"/>
                        </leaf>
                        </node>
                </node>

            </node>
        </r2dtotree>
    </np:fmtxt>
</wdgt>

<!-- Another example -->

<wdgt xmlns:np="urn:TI.Notepad" type="TI.Notepad" ver="2.0">
 <np:mFlags>1024</np:mFlags>
 <np:value>3</np:value>
 <np:fmtxt>
     <r2dtotree>
         <node name="1doc">
             <node name="1para">
                 <node name="1rtline">
                     <leaf name="1word">L1normal</leaf>
                 </node>
             </node>
             <node name="1para">
                 <node name="1rtline">
                     <leaf name="1word">L2</leaf>
                     <leaf name="1keyword">gras</leaf>
                 </node>
             </node>
             <node name="1para">
                 <node name="1rtline">
                     <leaf name="1word">L3</leaf>
                     <leaf name="1subhead">italique</leaf>
                 </node>
             </node>
             <node name="1para">
                 <node name="1rtline">
                     <leaf name="1subhead">L4</leaf>
                     <leaf name="1word">blabla</leaf>
                 </node>
             </node>
             <node name="1para">
                 <node name="1rtline">
                     <leaf name="1title">L5</leaf>
                     <leaf name="1title">souligné</leaf>
                 </node>
             </node>
             <node name="1para">
                 <node name="1rtline">
                     <leaf name="1word">L6</leaf>
                     <leaf name="1supersc">up</leaf>
                 </node>
             </node>
             <node name="1para">
                 <node name="1rtline">
                     <leaf name="1word">L7</leaf>
                     <leaf name="1subscrp">down</leaf>
                 </node>
             </node>
             <node name="1para">
                 <node name="1rtline">
                     <leaf name="1word">L8</leaf>
                     <leaf name="1word">on</leaf>
                     <leaf name="1title">mixxe|xx|</leaf>
                     <leaf name="1keyword">
                         <cursor index="0"/>eeee</leaf>
                     <leaf name="1keyword">==></leaf>
                     <leaf name="1keyword">fail</leaf>
                 </node>
             </node>
             <node name="1para">
                 <node name="1rtline">
                     <leaf name="1word">Et</leaf>
                     <leaf name="1word">ci</leaf>
                     <leaf name="1word">on</leaf>
                     <leaf name="1word">fait</leaf>
                     <leaf name="1word">ça?:</leaf>
                     <leaf name="1word"></leaf>
                     <leaf name="1keyword">11</leaf>
                     <leaf name="1title">22</leaf>
                     <leaf name="1keyword">11</leaf>
                 </node>
             </node>
         </node>
     </r2dtotree>
 </np:fmtxt>
</wdgt>
```