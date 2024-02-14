# Parametric Construction Kit

This week, we were challenged to laser cut a series of objects that could be used as a construction kit. I designed the kit in Fusion 360 where I could take advantage of parametric designing, defining and using variables to rapidly modify the model. For this design, I was inspired by <a href="https://fabacademy.org/2020/labs/charlotte/students/elaine-liu/">**Elaine Liu's**</a> creation.

## Designing

<table>
    <tr>
        <td><b>Parameter Name</b></td>
        <td><b>Description</b></td>
    </tr>
    <tr>
        <td>tabLength</td>
        <td>The length that each tab hole juts into the polygon</td>
    </tr>
    <tr>
        <td>materialThickness</td>
        <td>The thickness of the cardboard, used for determining the width of the tabs</td>
    </tr>    
    <tr>
        <td>filletVal</td>
        <td>The size of the fillet for each tab joint</td>
    </tr>    
    <tr>
        <td>connectorLength</td>
        <td>The length of the L-shape connectors</td>
    </tr>
        <tr>
        <td>connectorWidth</td>
        <td>The width of the L-shape connectors</td>
    </tr>
</table>

#### Hexagon
To start designing the hexagonal component, I created a polygon with six sides using the circumscribed polygon tool.

<center>
<img src="../../../pics/week3/hexagon.jpg" alt="Hexagon" width="250"/>
</center>

I then created a tab on one side of the hexagon. The tab was tabLength deep and materialThickness wide. Both sides were filleted by filletVal.

<center>
<img src="../../../pics/week3/singleTab.jpg" alt="tab" width="200"/>
</center>

I then used Fusion's Circular Pattern tool to repeat the tab on each side of the hexagon. I repeated the tab for a total of 6 tabs and centered the pattern at the center of the hexagon.

<center>
<img src="../../../pics/week3/circlePattern.jpg" alt="Circular pattern" width="400"/>
</center>

#### Square

I first used the center rectangle tool to create a square. Each side of the square must be the same length as the length of a side of the hexagon to fit together properly. To find the side of the hexagon, you could simply create a line on the side in Fusion360 which would return the measurement. However, I chose to find this value mathematically.

<center>
<table>
    <tr>
        <td><img src="../../../pics/week3/hexagonSideCalculation.jpg" width="335"/></td>
        <td><img src="../../../pics/week3/square.jpg" width="335"/></td>
    </tr>
</table>
</center>

After creating the basic square outline, I created a tab at the midpoint of the top edge of the square. The tab is tabLength deep and materialThickness wide, with a fillet of filletVal.

<center>
<img src="../../../pics/week3/squareTab.jpg" alt="Square with tab" width="400"/>
</center>

I then used the circular pattern tool to create a total of 4 tabs on the square, with the center point of the square as the centerpoint of the circular pattern. This created a tab at the midpoint of each side of the square.

<center>
<img src="../../../pics/week3/patternedSquare.jpg" alt="Square with all 4 tabs" width="300"/>
</center>

#### Connector

I started out by creating a rectangle that had dimensions connectorLength by connectorWidth.

<center>
<img src="../../../pics/week3/connectorRectangle.jpg" alt="Connector Rectangle" width="300"/>
</center>

I then created another rectangle with the same dimensions at a 120 degree angle to the first rectangle (each interior angle of a hexagon is 120 degrees).

<center>
<img src="../../../pics/week3/connectorBody.jpg" alt="Connector Body" width="300"/>
</center>

I then created a tab on the top part of the connector. The creation process is similar to making the tab on the hexagon. The tab is tabLength deep and materialThickness wide, with fillets of filletVal.

<center>
<img src="../../../pics/week3/connectorOneTab.jpg" alt="Connector with single tab" width="250"/>
</center>

I then mirrored the tab across the centerline of the connector.

<center>
<img src="../../../pics/week3/mirroredTab.jpg" alt="Connector with both tabs" width="500"/>
</center>

<br>

## Assembly

I first exported the Fusion360 file as a DXF and opened it in CorelDraw. In CorelDraw, I duplicated my designs until I had 8 hexagons, 6 squares, and 24 connectors and sent it to Epilog Engraver. Unfortunately, I miscalculated and was 12 connectors short, so I had to cut another 12 connectors afterwards.

<table>
    <tr>
        <td><img src="../../../pics/week3/corel.jpg" width="450"/></td>
        <td><img src="../../../pics/week3/epilog.jpg" width="450"/></td>
    </tr>
</table>

I then sent the design to the laser cutter and cut the cardboard.
<center>
<video muted width="500" height="300" controls><source src="../../../pics/week3/cuttingTimeLapse.mp4" type="video/mp4" /></video>
</center>

After pushing the pieces together, here is the final tetradecahedron.

<center>
<img src="../../../pics/week3/final.jpg" alt="Assembled tetradecahedron" width="500"/>
</center>