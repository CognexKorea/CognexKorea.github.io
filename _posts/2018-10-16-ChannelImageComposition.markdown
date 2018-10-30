---
layout: post
title:  "Python-OpenCV / How to Compose Channel Images"
date:   2018-10-16 22:18:00
author: Alex Choi
categories: Image-Manipulator
---

# How to Compose Channel Images using Python-OpenCV

Sometimes we need to set up various lightings to see defects better (Certain defects like scratch, dent, etc. are a sort of directional).

For instance, each image acquired from four different light directions can be fed into four channles of an image with which ViDi can analyze.

In this post I'm going to introduce how to compose (or mix) up to 4 channels into one image using Python-OpenCV as a preprocessor of ViDi analysis.

You can down load Python source from [this Github Repository](https://github.com/gchoi/Python-OpenCV-Channel-Image-Composition).

------
## Development Environment
* Python : ver. 3.5+
* [NumPy](https://pypi.org/project/numpy/) : ver. 1.14.5+
* [Python-OpenCV](https://pypi.org/project/opencv-python/) (cv2) : ver. 3.4.3+

If you are using Anaconda as Python management system then you can install Python-OpenCV package using [anaconda cloud](https://anaconda.org/conda-forge/opencv).
Also you can install it using `pip` command (Refer to [this site](https://pypi.org/project/opencv-python/)).

------
## Method
Let us assume that we have four images for the same object acquired from different lighting directions.

<img src="{{ site.baseurl }}/assets/posts/2018-10-15-ImageComposition/01.png">

Then each image is fed into the channel one by one, for example, R, G, B and A - the composition images must support 4 channels i.e. PNG, TIF, etc.

<img src="{{ site.baseurl }}/assets/posts/2018-10-15-ImageComposition/02.png">

------
## Code

Python code that composes channels is as follows:

{% highlight python %}
# -*- coding: utf-8 -*-
"""
Created on Tue Apr 17 22:51:38 2018
@ author: alex choi
@ purpose: composition of multi-channels from different lighting angles
@ note: input image filename convention:
       any image file format would be ok, but filename should be as following
       1-1.bmp, 1-2.bmp, 1-3.bmp, 1-4.bmp
       2-1.bmp, 2-2.bmp, 2-3.bmp, 2-4.bmp
       3-1.bmp, 3-2.bmp, 3-3.bmp, 3-4.bmp
       ...
       maximum 4 channels allowed
"""

#########################################################################
#      USER DEFINITION
########################################################################
## define input & output path
# @input path : where your input channel images
# @output path : where to put your composition images
NUM_LIGHT_DIRECTION = 4  # number of lighting directions
INPUT_IMAGE_PATH = './SourceImages'
OUTPUT_IMAGE_PATH = './ResultImages'
OUTPUT_IMAGE_NAME = 'ChannelMixed'
OUTPUT_IMAGE_FORMAT = 'bmp'
LIGHT_SELECTION = [1, 2, 3, 4]
#########################################################################

## import libraries
import os, errno
import sys
from os import walk
import numpy as np
import cv2

#########################################################################
# @function : main
#########################################################################
def main():
    if(len(LIGHT_SELECTION) > 4):
        sys.exit()

    ## create output path if not exist
    try:
        os.makedirs(OUTPUT_IMAGE_PATH)
    except OSError as e:
        if e.errno != errno.EEXIST:
            raise

    ## file list from input path
    flist = []
    for (dirpath, dirnames, filenames) in walk(INPUT_IMAGE_PATH):
        flist.extend(filenames)
        break

    ## initialize image mat
    tmp = cv2.imread(os.path.join(INPUT_IMAGE_PATH, flist[0]))
    img = np.zeros(shape=[tmp.shape[0], tmp.shape[1], len(LIGHT_SELECTION)], dtype=int)

    ## generate channel mixed images
    for filename in flist:
        tmp = filename.split("-")
        fileIdx = tmp[0]

        chNo = tmp[1].split(".%s" % (OUTPUT_IMAGE_FORMAT))[0]

        fileIdx = int(fileIdx)
        chNo = int(chNo)

        if(chNo in LIGHT_SELECTION):
            idx = LIGHT_SELECTION.index(chNo)
            tmp = cv2.imread(os.path.join(INPUT_IMAGE_PATH, filename))
            tmp = cv2.cvtColor(tmp, cv2.COLOR_BGR2GRAY)
            img[:,:,idx] = tmp


        if(chNo == NUM_LIGHT_DIRECTION):
            fname = os.path.join(OUTPUT_IMAGE_PATH, "%s-%03d.png" % (OUTPUT_IMAGE_NAME, fileIdx))
            cv2.imwrite(fname, img)
            print("Saved : %s" % (fname))


if __name__ == "__main__":
    main()
{% endhighlight %}

------
## Usage
Usage is very simple. You should just care about `User Definitions`.

```
NUM_LIGHT_DIRECTION = 4  # number of lighting directions
INPUT_IMAGE_PATH = './SourceImages'
OUTPUT_IMAGE_PATH = './ResultImages'
OUTPUT_IMAGE_NAME = 'ChannelMixed'
OUTPUT_IMAGE_FORMAT = 'bmp'
LIGHT_SELECTION = [1, 2, 3, 4]
```

While the user-defined variables are thought to be self-explanatory, I want to explain more about `LIGHT_SELECTION`.

 `LIGHT_SELECTION` is the sequence of the image selection that you can choose from. You can select 1, 2, 3 or 4 images and set the sequence of the images.

 For instance, `LIGHT_SELECTION = [3, 1, 4, 2]` means:
 * 3rd input image fed into 1st channel of output image.
 * 1st input image fed into 2nd channel of output image.
 * 4th input image fed into 3rd channel of output image.
 * 2nd input image fed into 4th channel of output image.

Or, `LIGHT_SELECTION = [3, 1, 4]` means:
* 3rd input image fed into 1st channel of output image.
* 1st input image fed into 2nd channel of output image.
* 4th input image fed into 3rd channel of output image.

------
## Examples
I give you some examples.

<img src="{{ site.baseurl }}/assets/posts/2018-10-15-ImageComposition/03.png">

<img src="{{ site.baseurl }}/assets/posts/2018-10-15-ImageComposition/04.png">
