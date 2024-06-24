# 3D Printing

## Assignment

- Design and 3D print an object that could not be made through subtractive manufacturing

## Designing

I had to create an object that could not be made through subtractive manufacturing. I decided to make a chain with interlocking links, which cannot be made through subtractive manufacturing because of its internal spaces. 

To design the chain, I first created a cube and created a sketch on its face. I then created a smaller rectangle on the sketch and extruded it into the cube as a hole.

<center>
<table>
    <tr>
        <td><img src="../../../pics/week5/cube.jpg" width="335"/></td>
        <td><img src="../../../pics/week5/extrude.jpg" width="335"/></td>
    </tr>
</table> 
</center>

After filleting the object, the first chain link was done. I then duplicated the link next to itself.

<center>
<img src="../../../pics/week5/duped.jpg" height="400" width="400"/>
</center>

Next, I copied the original link and rotated it 90 degrees. I then moved the pasted object to be positioned inside of the two other links.

<center>
<img src="../../../pics/week5/horizontalLink.jpg" height="400" width="400"/>
</center>

I then duplicated the horizontal chain link and added one more vertical chain link to create the final design.

<center>
<img src="../../../pics/week5/fusionChain.jpg" height="400" width="400"/>
</center>

## Printing

I first exported the design from Fusion360 into PrusaSlicer as a STL, and sliced the design.

<center>
<img src="../../../pics/week5/prusaChain.jpg" height="400" width="400"/>
</center>

My configuration for the slicing were mostly defaults. I used a 15% infill with a 0.2mm layer height (Prusa defaults). However, I wanted to use organic supports so that it would be easier to take out and the overall print wouldn't be affected much if the supports didn't cleanly come off (thankfully they did!). To generate organic supports in PrusaSlicer, I navigated to the Print Settings -> Support material menu, where I enabled supports and chose the "Organic" generation style.

<center>
<img src="../../../pics/week5/organicSupports.jpg" height="400" width="400"/>
</center>


Next, I needed to send the file to my 3D printer. I clicked the export button in the bottom right hand side of PrusaSlicer.

<center>
<img src="../../../pics/week5/export.jpg" height="400" width="400"/>
</center>

This opens up a dialog box where I can save the file to my local system. I saved the file to a USB drive, which I then plugged into the printer. I then set off the print.

<center>
<img src="../../../pics/week5/saving.jpg" height="400" width="400"/>
</center>

Because I wanted the chain links to interlock but also to be able to move around, the links were not directly touching each other in the GCODE design as to not stick them together. As such, I used supports to hold up parts of the chain.

<center>
<img src="../../../pics/week5/chainWithSupports.jpg" alt="Printed chain with supports" height="400" width="400"/>
</center>

After taking off the supports, my chain was ready.

<center>
<table>
    <tr>
        <td><img src="../../../pics/week5/chainReg.jpg" width="335"/></td>
        <td><img src="../../../pics/week5/bendyChain.jpg" width="335"/></td>
    </tr>
</table> 
</center>

