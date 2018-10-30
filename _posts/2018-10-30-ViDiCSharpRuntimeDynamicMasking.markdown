---
layout: post
title:  "ViDi / C# Runtime Dynamic Masking"
date:   2018-10-29 10:25:00
author: Alex Choi
categories: Deep-Learning
---

In this post I'd like to introduce how to implement "Runtime Dynamic Masking" using C# API.

Please click [here](https://cognexcorporation-my.sharepoint.com/:u:/g/personal/alex_choi_cognex_com/EcheH773qtVHr8GLLq3zoBQBl8Og2m_Zb21__OmKBYabTA?e=kcDD28) to download the Visual Studio Solution.

------
## Motivation
In the previous post, I've introduced [Dynamic Masking of Training Version](https://cognexkorea.github.io/deep-learning/2018/04/24/DynamicMasking.html) and I've been curious about how to implement the same thing in Runtime mode.

As you might excercise with ViDi Advanced Tutorial #4 - `Advanced Tutorial 4 - Dynamical Masking`, there's the better performance of Green Classification Tool with dynamic masking using Red Tool.

Using the Tutorial Workspace I separate two different Runtime Workspaces, one is for without mask and the other with mask and eventaully I wanted to compare the results from my implementation of Runtime dynamic masking and that from ViDi Production mode.

------
## Toolchain
#### Classification without Dynamic Masking
Only Green tool was used to classfy batteries which has background unmasked.

<img src="{{ site.baseurl }}/assets/posts/2018-10-30-ViDiCSharpRuntimeDynamicMasking/01.png">

As shown in Confusion Matrix there are lots of mismatches (22).

Since whole pixels are included for classification background and shadows, which may be undesirable for our classification problem, can take effect on the results.

<br/>
#### Classification with Dynamic Masking
Then how can we remove backgrounds effectively? This is why dynamic masking comes into effect.

So the toolchain becomes like below:

<img src="{{ site.baseurl }}/assets/posts/2018-10-30-ViDiCSharpRuntimeDynamicMasking/02.png">

The number of mismatches decreases from 22 to 7.

Red tool / supdervised mode finds only battery regions (treated as defect areas) and green tool sets the ROI with the battery areas from Red tool (the mask was inverted).

<img src="{{ site.baseurl }}/assets/posts/2018-10-30-ViDiCSharpRuntimeDynamicMasking/03.png">

Since each of image has its own mask we call it "dynamic mask".

------
## Implementation
Note that I separated the ViDi Runtime workspaces into masked and unmasked version because currently ViDi API for runtime only supports when the tool is located at the root to which dynamic masking is applied.

So there are 2 runtime workspaces under the path,  `Resources/vidi-runtime-workspace` in Visual Studio project folder:

* Advanced Tutorial 4 - Dynamical Masking (solution)_21_unmask.vrws: Green tool trained WITHOUT masks
* Advanced Tutorial 4 - Dynamical Masking (solution)_21_mask.vrws: Green tool trained WITH masks

<br/>
#### Global User Definitions

{% highlight csharp %}
static class Globals
{
    // path where ViDi runtime workspace exists
    public const string VRWS_PATH = "..\\..\\Resources\\vidi-runtime-workspace";

    // flag for using dynamic mask
    public const bool bUseMask = false;

    // ViDi Runtime workspace filename without ".vrws"
    public const string VRWS_MASK_FILENAME = "Advanced Tutorial 4 - Dynamical Masking (solution)_21_mask";
    public const string VRWS_UNMASK_FILENAME = "Advanced Tutorial 4 - Dynamical Masking (solution)_21_unmask";

    // stream name
    public const string STREAM_NAME = "default";

    // Blue Read tool name
    public const string TOOL_NAME = "Classify";

    // test image path
    public const string TEST_IMG_PATH = "..\\..\\Resources\\test-images";

    // mask image
    public const string MASK_IMG = "..\\..\\Resources\\mask-images";
}
{% endhighlight %}

In the code above there are bunch of global variables:

|   Global Variables   |                   Description                   |
|:---------------:|:----------------------------------------:|
|  <font face="Consolas">VRWS_PATH</font> | Path where ViDi runtime workspace is located          |
|  <font face="Consolas">bUseMask</font> | Flag for mask to used or not            |
| <font face="Consolas">VRWS_MASK_FILENAME</font> | ViDi runtime workspace trained with mask                |
|   <font face="Consolas">VRWS_UNMASK_FILENAME</font>   | ViDi runtime workspace trained without mask       |
|   <font face="Consolas">STREAM_NAME</font>   | Stream name                         |
|    <font face="Consolas">TOOL_NAME</font>    | Tool name    |
| <font face="Consolas">TEST_IMG_PATH</font>     | Path where test images are located |
| <font face="Consolas">MASK_IMG_PATH</font>     | Path where mask images are located |

<br/>
#### Core Code Lines

To make long story short I'd like to introduce just core part of the code lines in `Main` procedure. So please refer to the Visual Studio solution file to look at the whole code.

{% highlight csharp %}
ISample sample = stream.CreateSample(image);

IManualRegionOfInterest mRoi = (IManualRegionOfInterest)tool.RegionOfInterest;
mRoi.External = true;
System.Windows.Size my_size = new System.Windows.Size(0.0, 0.0);
System.Windows.Media.Matrix my_mat = new System.Windows.Media.Matrix(1.0, 0.0, 0.0, 1.0, 0.0, 0.0);
sample.AddView(tool, my_size, my_mat, maskImage);

// processing
sample.Process(tool);
{% endhighlight %}

`sample.AddView()` method add `View` with a mask image with specified size and matrix into `tool` and then `sample.Process()` processes the image with masking.

------
## Verification
Verification was done by comparing between the results from ViDi production mode and those from C# API Runtime code.

<img src="{{ site.baseurl }}/assets/posts/2018-10-30-ViDiCSharpRuntimeDynamicMasking/04.png">

Five images are randonly chosen among mismatches to compare with and without masks for both runtime code and productino mode in ViDi.

The mask images are exported manually for the corresponding images (Edit Mask > Export).

<img src="{{ site.baseurl }}/assets/posts/2018-10-30-ViDiCSharpRuntimeDynamicMasking/05.png">

The name of each file contains its category. For example the file `Cat12-00007.png` belongs to `Cat12` class.

As we can see from the table above, if we round off the numbers of `Dynamic Masking using Runtime API` to the nearest hundredths the same results between two cases, which means dynamic masking turned out to be proven.
