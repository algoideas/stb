# Font character oversampling for rendering from atlas textures

TL,DR: Run oversample.exe on a windows machine to see the
benefits of oversampling. Type the name of a .ttf file on
the command-line, or it will look for Arial in the Windows
font directory.

## About oversampling

A common strategy for rendering text is to cache character bitmaps
and reuse them. For hinted characters, every instance of a given
character is always identical, so this works fine. However, stb_truetype
doesn't do hinting.

For anti-aliased characters, you can actually position the characters
with subpixel precision, and get different bitmaps based on that positioning
if you re-render the vector data.

However, if you simply cache a single version of the bitmap and
draw it at different subpixel positions with a GPU, you will get
either the exact same result (if you use point-sampling on the
texture) or linear filtering. Linear filtering will cause a sub-pixel
positioned bitmap to blur further, causing a visible desharpening
of the character. (And, since the character wasn't hinted, it was
already blurrier than a hinted one would be, and not it gets even
more blurry.)

You can avoid this by caching multiple variants of a character which
were rendered independently from the vector data. For example, you
might cache 3 versions of a char, at 0, 1/3, and 2/3rds of a pixel
horizontal offset, and always require characters to fall on integer
positions vertically.

When creating a texture atlas for use on GPUs, which support bilinear
filtering, there is a better approach than caching several indepdent
positions, which is to allow lerping between the versions to allow
finer subpixel positioning. You can achieve these by interleaving
each of the cached bitmaps, but this turns out to be mathematically
equivalent to a simpler operation: oversampling and prefiltering the
characters.

So, setting oversampling of 2x2 in stb_truetype is equivalent to caching
each character in 4 different variations, 1 for each subpixel position
in a 2x2 set.