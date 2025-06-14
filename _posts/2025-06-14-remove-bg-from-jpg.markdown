---
layout: post
title:  "Remove white background from jpg SNotes export"
date:   2025-06-14 22:01:00 +0100
categories: how to python scripting
tags: ["how to", "python", "scripting"]
---

**Today we have a completely different topic!** - as you can tell by last post's date, I have been quite busy at work and on my spare time, leaving me no time to deep dive into topics I'm interested in. While I hope to get my hands on more interesting/challenging tasks and projects at work (at least more interesting as the ones I usually take care of), during my spare time I'm kind of struggling with a double-move which will leave me homeless for a month :D

But that's not happening yet and after stapling some cardboard boxes today, I ran into a little challenge.

## The first challenge

Importing/exporting notes between Samsung Notes and Goodnotes might seem quite simply: export as PDF from the one side, import from PDF on the other side, continue editing your notes. **But** the problems start when you want to modify anything on the imported note, such as:

- Changing page background layouts
- Moving one text block within the page or within pages (using the lasso tool for instance)

These kind of apps turn your hardwritten strokes and doodles into **ink data** - which contains information about the pen you're using, the ink color, etc. When exporting as PDF, all this ink data gets lost, which leaves us with a dull, non-editable file. And no matter what people said on Reddit 5 years ago - renaming your exported sdocx (Samsung Notes format) into a .zip, unpacking it and looking for contents on the /media folder won't give you anything usable.

Other export options wouldn't really suit my use case, as converting hard writing into text doesn't work great (specially with math and chemistry formulas). Exporting from one proprietary format into the other wasn't any useful either.


## The next problem

But what if you export pages of your Samsung Notes as pictures? If you first modify the page layout, you will remove any lines/squares from the background, leaving you with a plain .jpg with white background and colored strokes. Afterwards you can simply paste the image directly into your second note app (Goodnotes, in this case), and you're good to go.

But what if you want to use a nice page layout on Goodnotes? A white backgroud image doesn't look that nice on a light-colored lined page. The easiest would be removing the white background from the exported .jpg file, leaving a nice .png with a transparent background.


## The solution

There are many ways of turning a white background into a transparent one:

- Photoshop (or any other image editing software)
- Sites like remove.bg will do it for free (no registration needed, although you do need to register and use a paid plan to get nice resolutions)

But if you still want to do it by yourself, without depending on any proprietary softare or third-party sites, you can use a Python script like this one:

```console
import sys
from PIL import Image

def make_white_background_transparent(image_path, output_path):
    # Open the image and convert to RGBA
    img = Image.open(image_path).convert("RGBA")
    datas = img.getdata()

    newData = []
    for item in datas:
        # If pixel is pure white, make it transparent
        if item[0] > 240 and item[1] > 240 and item[2] > 240:
            newData.append((255, 255, 255, 0))
        else:
            newData.append(item)

    img.putdata(newData)
    img.save(output_path, "PNG")
    print(f"Saved output to {output_path}")

if __name__ == "__main__":
    if len(sys.argv) != 3:
        print("Usage: python remove_white_bg.py input.jpg output.png")
        sys.exit(1)

    input_file = sys.argv[1]
    output_file = sys.argv[2]

    make_white_background_transparent(input_file, output_file)

```

For this you only need **Python and a virtual environment (venv)** - you can use [this tutorial](https://www.freecodecamp.org/news/how-to-setup-virtual-environments-in-python/) from Stephen Sanwo on freeCodeCamp to install and get your venv running. The script basically takes an input image and examines it pixel by pixel using an RGBA schema. If the pixel color is almost white (red, green and blue values greater than 240), its opacity will be turn into transparent. All pixels (either transformed or not) are saved into an array, which will then be used to render a PNG image.

To use the script:

* Activate your virtual environment: ```source env/bin/activate```
* Install [pillow](https://pypi.org/project/pillow/), as this library is imported on runtime: ```pip install pillow```
* I named my script deletebg.py - it takes an **input file** and a **name for an output file** as arguments:

```console
python deletebg.py input.jpg output.png
```

* The script also has an output text which will let you know when it's done.

![Script run on cmd](/assets/img/deletebg_run.png)

**That's it!** Now you have a clean .png with a transparent background that can be pasted into any Goodnotes' page layout.