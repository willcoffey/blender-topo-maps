Purpose of this repo is to create a method for making topographical, 3D printable, maps in blender

# Initial process

```
ProductName:		macOS
ProductVersion:		14.5
BuildVersion:		23F79

Version: ImageMagick 7.1.1-38 Q16-HDRI aarch64 22398 https://imagemagick.org
Copyright: (C) 1999 ImageMagick Studio LLC
License: https://imagemagick.org/script/license.php
Features: Cipher DPC HDRI Modules OpenMP(5.0) 
Delegates (built-in): bzlib fontconfig freetype gslib heic jng jp2 jpeg jxl lcms lqr ltdl lzma openexr png ps raw tiff webp xml zlib zstd
Compiler: gcc (4.2)

Blender 4.2.2 LTS
```

## Image Processing
- Download height data as `.tif` tiles. They also have point cloud data that could be interesting 
  but I didn't look at.
    [MassGIS Data Lidar Terrain Data](https://www.mass.gov/info-details/massgis-data-lidar-terrain-data)
  They will likely look all white, either due to 32-bit floats not being supported in many viewers, 
  or the info being shunted into the very light shades.

- Rename files to get order correct for combining. For example: (this is partially a relic of when 
  I was trying to use montage)
    ```sh
    cp ../be_19TCG318690.tif  00.tif
    cp ../be_19TCG319690.tif  01.tif
    cp ../be_19TCG321690.tif  02.tif
    cp ../be_19TCG322690.tif  03.tif
    cp ../be_19TCG324690.tif  04.tif
    cp ../be_19TCG318689.tif  05.tif
    cp ../be_19TCG319689.tif  06.tif
    cp ../be_19TCG321689.tif  07.tif
    cp ../be_19TCG322689.tif  08.tif
    cp ../be_19TCG324689.tif  09.tif
    cp ../be_19TCG318687.tif  10.tif
    cp ../be_19TCG319687.tif  11.tif
    cp ../be_19TCG321687.tif  12.tif
    cp ../be_19TCG322687.tif  13.tif
    cp ../be_19TCG324687.tif  14.tif
    ```

- Combine using image magick. Note that `montage` does not work. It messes with the images and 
  loses data, took me a while to realize since tiles already appear white. use `magick` with 
  `+append` and `-append`. Needs to be done in stages, combine tiles into rows, then rows into grid.
    ```sh
    magick 0[0-4].tif +append row_0.tif
    magick 0[5-9].tif +append row_1.tif
    magick 1[0-4].tif +append row_2.tif
    magick row_[0-2].tif -append combined.tif
    ```
- Once combined, the height data can be normalized using `-auto-level` and the bit depth can be 
  changed via `-depth`. 
    ```sh
    magick combined.tif -auto-level -depth 8 combined.tif
    ```

- Once leveled, the tif needs to be turned into a square for the blender texture. You can get a 
  image details via `magick identify combined.tif`
    ```sh
    magick combined.tif -gravity north -background black -extent 15000x15000 square.tif
    ```

- Optionally resize via `magick  square.tif -resize 2560x2560 smaller.tif`
 
## Blender
from a new project

- delete the default cube using `x`
- Add a plane. `shift-a` then `mesh -> plane`
- Add modifier (blue wrench) `Generate -> Subdivision Surface`
    - set to simple
    - set levels to something like 6, more levels more details. See notes.
- Add modifier `Deform -> Displace`
    - Create new texture via `New` button
    - Show texture tab, right most button in texture row.
    - Open image, select the final `smaller.tif`
    - Click on blue wrench to go back to modifiers, adjust strength (.15) and mid level (0)

- On my computer, I can increase subdivision levels to 11 if I manually input it. This is a mesh
  that is 2048x2048.



# Notes
- I don't know why blender only lets you use square textures, messed with uv maps for a bit but it
  didn't work out.
- Resolution isn't great. subdivisions can be increased by using multiresolution modifier, but 12 
  caused issues on my machine. Also, it's seems inefficient to only be able to do factors of 2 and 
  squares.  There has to be a better way. Could do multiple tiles with multiple planes, but don't 
  want to have to click the same stuff 15 times. Multiple tiles is likely best approach, also 
  allows for non-square layouts.
- Could also look into point cloud or other 3D map data. Just look at the detail google maps has for
  3D models. I didn't look into it too much, but it seems like getting the data would be a pain.
  I did see one project where people used photogrammetry to take google maps screenshots and
  create a new 3D model. Would work well but god that sounds so roundabout.
- Could print negative to use as a mold for casting or electroforming.

# Todo
- Find a road network map in monochrome aligned with the same tiles as tif. Combine with existing 
  tif to lower roads deeper so they are well defined.
- Maybe make road network seperate object, print in multiple colors with the roads being different.
- Decide on map area
- Create 3D model of points of interest and add to map?
- Edit tif in photoshop or other to reduce water body levels, later fill with blue epoxy. Or just
  print in different color.
- Figure out how to get to STL. How to not print solid, etc.
