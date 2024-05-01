# Mold Designing

## Prusa

I first created my 3D mold design in Fusion.

<center>
<img src="../../../pics/week12/fusionDesign.jpg" width="750"/>
</center>

I then exported my design into PrusaSlicer for slicing.

<center>
<img src="../../../pics/week12/exportToPrusa.jpg" width="250"/>
</center>

Around this time, I was helping one of my lab instructors, <a href="https://fabacademy.org/2023/labs/charlotte/students/zack-budzichowski/">**Zack Budzichowski**</a>, set up <a href="https://connect.prusa3d.com/">**Prusa Connect**</a> for our lab's 3D printers. Prusa Connect allows a user to connect to a 3D printer completely remotely and start new prints and monitor its status. I decided to try out this new method to print my mold.

<center>
<img src="../../../pics/week12/prusaConnect.jpg" width="600"/>
</center>

I then uploaded and printed my file.

<center>
<img src="../../../pics/week12/prusa.jpg" width="350"/>
</center>

## Wax

In a previous engineering class, I had designed a heart-shaped wax mold as detailed <a href="https://richard-shan.github.io/edm2/lessons/milling/">**here**</a>. However, this was my first time working with milling and I ended up selecting the wrong toolpath and inverting the actual design when I milled it. I also didn't know about the simulate feature inside of Fusion.

As such, my failed design looked like this:

<center>
<video width="350" height="300" controls><source src="../../../pics/week12/failedSimulation.mp4" type="video/mp4" /></video>
</center>

However, I now knew that I needed to use a 2D toolpath instead of a 3D adaptive clear to cut out the heart shape that I originally wanted. My toolpath looked good as it actually cut in the correct spot this time.

To create the toolpath, I exited Fusion's "Design" mode and swapped to "Manufacture". There, I created a 2D Pocket toolpath and selected the heart-shaped depression in the mold. In my case, my roughing toolpath used a 1/16" Flat End Mill and the finishing pass used a 1/32" Ball End Mill.

<center>
<img src="../../../pics/week12/heartMoldFusion.jpg" width="500"/>
</center>

I then milled the file. The imperfections across the top of the wax are bubbles that arose when the wax was melted for reuse. The bubbles were not created by the milling process nor will they have any effect on the casting.

<center>
<img src="../../../pics/week12/wax.jpg" width="350"/>
</center>

## Resin

For my resin print, I decided to go with the same design as my Prusa print so I could compare the cast qualities of the Prusa vs resin print. However, I scaled the design down from a 5x5 inch print to 3x3 so I could save resin, which is a lot more expensive.

<center>
<img src="../../../pics/week12/fusionDesign.jpg" width="750"/>
</center>

I followed <a href="https://docs.google.com/document/d/1ZODfP5lM4_LH7BzHrylY2pK6lu1T3ovhMI_Vy-a4Bbk/edit">**this workflow**</a> for resin printing. I first exported the file out of Fusion and took it into Preform, the software that our lab uses for resin printing. I then set up the configuration for the material type and printer config. Here is a render of my resin print in PreForm.

<center>
<img src="../../../pics/week12/preform.jpg" width="350"/>
</center>

I then ran the job and cured the resin. Here is the final resin printed mold.

<center>
<img src="../../../pics/week12/resin.jpg" width="350"/>
</center>