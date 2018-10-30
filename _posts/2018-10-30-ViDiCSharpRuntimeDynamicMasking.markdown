---
layout: post
title:  "ViDi / C# Runtime Dynamic Masking"
date:   2018-10-29 10:25:00
author: Alex Choi
categories: Deep-Learning
---

# How to Implement C# Runtime Dyanamic Masking using ViDi API
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