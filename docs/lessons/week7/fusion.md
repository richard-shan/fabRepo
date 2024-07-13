# Fusion Designing

For this week's project, I decided I wanted to make a circular shelf. I wanted the shelf to have a tripod base and have 3 shelf layers where I can store things on.

I first created the 3D design of my shelf in Fusion.

<center>
<img src="../../../pics/week7/fusionShelf.jpg" alt="Shelf design in Fusion" width="350"/>
</center>

Each shelf board is 0.45 inches think, reflecting the thickness of the actual wood board. Because I plan to cut each shelf out on the same flat piece of wood, I can create a parameter called "woodThickness" and apply it to each shelf board. Additionally, because I wasn't exactly sure of the dimensions of my project during my initial design process, I created a variety of parameters that I can use to specify the shelf's total height, the offset of each shelf board, the size of each shelf board, the joint clearances, among others.

<center>
<img src="../../../pics/week7/parameters.jpg" alt="Shelf design parameters in Fusion" width="650"/>
</center>

I then used Fusion's ```Modify -> Arrange``` tool to arrange all of my components onto the flat XY plane. 

<center>
<img src="../../../pics/week7/fusionArrangement.jpg" alt="Fusion arrangement" width="350"/>
</center>

I then created a sketch on the XY plane and used the ```Create -> Include -> Intersect``` function to get the cross-section of my design where it intersects the flat plane.

<center>
<img src="../../../pics/week7/fusionSketch.jpg" alt="Fusion sketch" width="350"/>
</center>

Next, I selected my sketch from the explorer menu and right clicked it. I used the ```Save as DXF``` function to export the sketch as a DXF file.

<center>
<img src="../../../pics/week7/saveAsDXF.jpg" alt="Save As DXF menu" width="350"/>
</center>

I will now open up the file in Aspire to prepare it for cutting.