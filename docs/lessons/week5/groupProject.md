# Week 5 Group Project

## Group Assignment
For this week, our assignment as a group was to test the design rules for our 3D printer (a Original Prusa MINI+ printer). 

## Work Distribution

<center>
<table>
    <tr>
        <td><b>People</b></td>
        <td><b>Description</b></td>
    </tr>
        <td>Richard, David</td>
        <td>Printing</td>
    </tr>
    <tr>
        <td>Richard, David</td>
        <td>Documentation</td>
    </tr>    
    <tr>
        <td>Richard, David</td>
        <td>File preparations</td>
    </tr>    
    <tr>
        <td>Richard</td>
        <td>Printer Troubleshooting</td>
    </tr>
    <tr>
        <td>Richard</td>
        <td>Term Definitions</td>
    </tr>
    <tr>
        <td>David</td>
        <td>File Procurement</td>
    </tr>
    <tr>
</table>
</center>

## Term Definitions

<table>
    <tr>
        <th>Term</th>
        <th>Definition</th>
    </tr>
    <tr>
        <td>Layer Height</td>
        <td>Thickness of each print layer; lower heights yield finer resolution.</td>
    </tr>
    <tr>
        <td>Supports</td>
        <td>Temporary structures for stabilizing overhangs, removed post-printing.</td>
    </tr>
    <tr>
        <td>Infill</td>
        <td>Internal print structure; higher percentages increase strength and material use.</td>
    </tr>
    <tr>
        <td>Bed Adhesion</td>
        <td>Methods to ensure first layer adherence, like heated beds or adhesives.</td>
    </tr>
    <tr>
        <td>Raft</td>
        <td>Base layer for improving bed adhesion and leveling.</td>
    </tr>
    <tr>
        <td>Brim</td>
        <td>Thin perimeter layer to enhance base adhesion.</td>
    </tr>
    <tr>
        <td>Skirt</td>
        <td>Outline printed around the object for extruder priming.</td>
    </tr>
    <tr>
        <td>Extruder</td>
        <td>Feeds and melts filament for deposition.</td>
    </tr>
    <tr>
        <td>Nozzle</td>
        <td>Extrudes melted filament; size affects print resolution and speed.</td>
    </tr>
    <tr>
        <td>Build Plate</td>
        <td>Surface for print construction, often heated for better adhesion.</td>
    </tr>
</table>



## Characterization

We first downloaded a 3D printer torture test file from <a href="https://www.thingiverse.com/search?q=torture+test&page=1&category_id=73">**Thingiverse**</a> and sent it to PrusaSlicer. 

<center>
<img src="../../../pics/week5/gcodeRender.jpg" height="400" width="350"/>
</center>

At this point, we ran into some issues in sending our job to the printer. The 3D printer was not recognizing our files when we plugged in a USB drive, despite the drive working on multiple computers that we used for testing. We managed to find a workaround by uploading the print to Octoprint which allowed us to remotely connect and control our 3D printers via a local network connection. We then ran the job.

<center>
<img src="../../../pics/week5/printBedStressTest.jpg" height="400" width="400"/>
</center>

The printer overall performed surprisingly well on all the tests. It had some trouble with the thin, tall pillars but the general shape is still recognizable. Notably, the printer flawlessly executed the overhang test, printing up to 80 degrees from the last support point.

<center>
<table>
    <tr>
        <td><img src="../../../pics/week5/stressTestMain.jpg" width="335"/></td>
        <td><img src="../../../pics/week5/angleTest.jpg" width="335"/></td>
    </tr>
</table> 
</center>

## Individual Contribution

I was responsible for defining terms, preparing the file for printing, troubleshooting printer connectivity issues, executing the print job, and documenting our work.

## File Downloads

Our group's files for this week can be downloaded [**here**](../../../files/week5/richardDavidWeek5.zip).