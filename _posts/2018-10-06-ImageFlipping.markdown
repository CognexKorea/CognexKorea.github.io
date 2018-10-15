---
layout: post
title:  "OpenCV-C# / Image Flipping Solution"
date:   2018-10-06 00:30:00
author: Alex Choi
categories: Image-Manipulator
---

## Github Link
* [https://github.com/gchoi/Image-Flipping-using-OpenCV](https://github.com/gchoi/Image-Flipping-using-OpenCV)
* [Execution Binary](https://github.com/gchoi/Image-Flipping-using-OpenCV/blob/master/Exceution.zip)

---
# Image Flipping using C# OpenCV
This is an Image Flipping solution using OpenCV in C#.

---
## Development Environment
* OS : Windows 10 x64
* IDE : Visual Studio 2015

---
## Usage
After compiling the sollution the exexcution file, `CSharp.OpenCV.Flip.exe`, is located under

`{$SOlutionDir}/{$ProjectDir}/bin/{$Configuration}/`

In a command line tool navigate to the path above and type the command as follows:

`> CSharp.OpenCV.Flip.exe arg1 arg2 arg3 arg4`

* `arg1`: Image Source Path
* `arg2`: Image Target Path
* `arg3`: Image Flipping-axis, X, Y or XY
  - X: Flip the image for X-axis
  - Y: Flip the image for Y-axis
  - XY: Flip the image for both axes
* `arg4`: Prefix of the created image flle name

---
## Example

`CSharp.OpenCV.Flip.exe "C:/SourceImagePath" "C:/TargetImagePath" "Y" "Flipped"`
