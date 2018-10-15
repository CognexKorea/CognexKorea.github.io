---
layout: post
title:  "C#-OpenCV / How to Blend Two Images"
date:   2018-10-15 00:30:00
author: Alex Choi
categories: Image-Manipulator
---

# Image Composition(Alpha Blending) using C# OpenCV
Currently ViDi API provides the function `Bitmap.Save()` to save "view" or "marking" images independently.

For example, the code below saves both images in the specified path:

{% highlight csharp %}
// retrieve the marking for the red tool called 'analyze'
IRedMarking redMarking = sample.Markings["Analyze"] as IRedMarking;

// gets the overlay image
IImage viewImage = redMarking.ViewImage(0);
IImage overlayImage = redMarking.OverlayImage(0);

// save images in the specfied path
viewImage.Bitmap.Save(savefolderpath + "\\" + view_image_filename + ".png", System.Drawing.Imaging.ImageFormat.Png);
overlayImage.Bitmap.Save(savefolderpath + "\\" + overlay_image_filename + ".png", System.Drawing.Imaging.ImageFormat.Png);
{% endhighlight %}

"view" image looks like:
<img src="{{ site.baseurl }}/assets/posts/2018-10-15-ImageComposition/view_image.png">

Also, "marking" image(with transparency channel) looks like:
<img src="{{ site.baseurl }}/assets/posts/2018-10-15-ImageComposition/overlay_image.png">

And we want a resultant image that shows the original "view" image with an overlay "marking" image on top of that:
<img src="{{ site.baseurl }}/assets/posts/2018-10-15-ImageComposition/composition_image.png">

Unfortunately, ViDi API does not provide any function that saves the composed images of "view" and "marking" images.

Thus using "marking" image in PNG format with the 4-channel color depth - as you know the 4th color channel has the transparency information called alpha depth - I'd like to share how to compose the images - "view" image as a underlying one and "marking" image as an overlay one.


---
## Github Link
You can download the source code from the Gituhub link below:
* [https://github.com/gchoi/CSharp.OpenCV.ImageComposition](https://github.com/gchoi/CSharp.OpenCV.ImageComposition)

---
## Usage
Just build the source in Visual Studio and run it with the modification of some source images or their paths.
