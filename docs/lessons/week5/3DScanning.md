# 3D Scanning

## Assignment

- 3D scan an object

## Scanning

As per recommendations from my instructors and previous years' Fab graduates, I used the app Polycam for my scanning. I decided to 3D scan a gaming microphone that was sitting in our lab.

<center>
<img src="../../../pics/week5/microphone.jpg" height="400" width="400"/>
</center>

As the microphone is a relatively complex object, I first moved it to an area with relatively little background clutter and good lighting. I put the microphone on a white clean table that was directly under a ceiling light and next to a window. As is visible in the video, this meant that my scan was well-lit and focused.

<center>
<img src="../../../pics/week5/location.jpg" height="400" width="400"/>
</center>

To use Polycam to generate a 3D model, I then took around 60 photos of the microphone from a multitude of different angles around the mic. After uploading the photos to Polycam, it generated a GLB file which I could view and convert to other formats. 

<center>
<video muted width="300" height="500" controls><source src="../../../pics/week5/render.mp4" type="video/mp4" /></video>
</center>

I then imported the .OBJ file into Fusion360, where I could view and modify the design.

<center>
<img src="../../../pics/week5/micFusion.jpg" height="400" width="400"/>
</center>

As is shown in the scanned model, the actual design is very rough. While shapes are roughly preserved, there is a lot of background that is scanned and there is no geometry in the actual model. However, this was a cool skill to learn and I might try this out in the future on other objects.

If I were to print this model, one major problem could be the scanned ground in the print. I could address this in a couple ways:

 - create a flat plane to intersect the microphone base in Fusion, and delete everything under it
 - create a rectangular box to intersect the microphone base in Fusion, and delete everything under it
 - manually edit the model's height coordinates and cut off anything underneath 0, thus deleting the lower layers
 - cut out the ground layers after slicing in PrusaSlicer
