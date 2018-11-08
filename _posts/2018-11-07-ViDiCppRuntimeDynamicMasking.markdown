---
layout: post
title:  "ViDi API / C++ Runtime Dynamic Masking"
date:   2018-11-07 23:25:00
author: Alex Choi
categories: Deep-Learning
---

[In the last post](https://cognexkorea.github.io/deep-learning/2018/10/29/ViDiCSharpRuntimeDynamicMasking.html) I introduced "Runtime Dynamic Masking" using C# API.
Now in this post I'd like to introduce how to implement "Runtime Dynamic Masking" using C++ API.

Please click [here](https://cognexcorporation-my.sharepoint.com/:u:/g/personal/alex_choi_cognex_com/EWdvX2zq0q1Ol0FO85PPSQIBDKF8AdnGFdv8Zq1NlXx8gw?e=gYk0dj) to download the Visual Studio Solution.

------
## Motivation
In the previous post, I've introduced [Dynamic Masking of Training Version](https://cognexkorea.github.io/deep-learning/2018/04/24/DynamicMasking.html) and I've been curious about how to implement the same thing in Runtime mode.

As you might excercise with ViDi Advanced Tutorial #4 - `Advanced Tutorial 4 - Dynamical Masking`, which shows a better performance of Green Classification Tool with dynamic masking using Red Tool.

Using the Tutorial Workspace I separate two different Runtime Workspaces, one is for without mask and the other with mask and eventaully I wanted to compare the results from my implementation of Runtime dynamic masking and that from ViDi Production mode.

------
## Toolchain
#### Classification without Dynamic Masking
Only Green tool was used to classfy batteries which have background unmasked.

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
#### Development Environment
* **IDE** : Visual Studio v14(2015)
* **ViDi** : ver.3.1.1.10701
* **OpenCV** : [4.0.0-beta](https://opencv.org/releases.html)(Pre-release)

<br/>
#### Concepts
<img src="{{ site.baseurl }}/assets/posts/2018-11-07-ViDiCppRuntimeDynamicMasking/01.png">

<br/>
#### Folder Structure
The folder structure of my Visual Studio Solution is as follows:
* **Visual Studion Solution Root**
  - **ViDi.Cpp.Runtime.Dynamic.Masking** : Visual Studio Project path which contains .cpp sources and headers
  - **Resources** : Necessary files are located here for runtime processing
    - `vidi-runtime-workspace` : ViDi rutime workspaces (with & without masking)
    - `test-images` : Test images from ViDi official basic tutorial - 4
    - `mask-images` : Mask images exported from Edit mask tool of ViDi GUI Suite
  - **OpenCV** : OpenCV is used to combine the original, mask and overlay graphics images
  - **Results** : This path has the following three types of files
    - `.xml` : XML files that contain processing results - score, tags, etc.
    - `COMBINED-XXXX.png` : Combined(blended) images - original + mask + overlay graphics
    - `OVERLAY-XXXX.png` : Overlay graphics images


------
## Code Explanation

I would brifly look through the essential code lines and if anyone wants to know the detail please refer to the whole code included in the later part of this post.

#### Global User Definitions

{% highlight c++ %}
////////////////////////////////////////////////////////////
// DEFINE GLOBAL VARIABLES
////////////////////////////////////////////////////////////
const std::string VRWS_PATH("../Resources/vidi-runtime-workspace/");
const std::string VRWS_MASK_FILENAME("Advanced Tutorial 4 - Dynamical Masking (solution)_21_mask");
const std::string VRWS_UNMASK_FILENAME("Advanced Tutorial 4 - Dynamical Masking (solution)_21_unmask");

const std::string RESULT_PATH = "../Results/";

const char* STREAM_NAME = "default";
const char* TOOL_NAME = "Classify";
const char* SAMPLE_NAME = "my_sample";

const std::string TEST_IMG_PATH("../Resources/test-images");
const std::string MASK_IMG_PATH("../Resources/mask-images");

// flag for using dynamic mask
const bool bUseMask = true;
{% endhighlight %}

In the code above there are bunch of global variables defined:

|   Global Variables   |                   Description                   |
|:---------------:|:----------------------------------------:|
|  <font face="Consolas">VRWS_PATH</font> | Path where ViDi runtime workspace is located          |
| <font face="Consolas">VRWS_MASK_FILENAME</font> | ViDi runtime workspace trained with mask                |
|   <font face="Consolas">VRWS_UNMASK_FILENAME</font>   | ViDi runtime workspace trained without mask       |
|   <font face="Consolas">RESULT_PATH</font>   | ViDi runtime workspace trained without mask       |
|   <font face="Consolas">STREAM_NAME</font>   | Stream name                         |
|    <font face="Consolas">TOOL_NAME</font>    | Tool name    |
|    <font face="Consolas">SAMPLE_NAME</font>    | Sample name    |
| <font face="Consolas">TEST_IMG_PATH</font>     | Path where test images are located |
| <font face="Consolas">MASK_IMG_PATH</font>     | Path where mask images are located |
|  <font face="Consolas">bUseMask</font> | Flag for mask to used or not            |

<br/>
#### Initialize Device Tools
{% highlight c++ %}
// initialize the libary to run with one GPU per tool
vidi_initialize(VIDI_GPU_SINGLE_DEVICE_PER_TOOL, "");
{% endhighlight %}

First thing to do for runtime processing is initializing device(GPU).

<br/>
#### Open Runtime Workspace
{% highlight c++ %}
// define runtime workspace path
std::string strVrwsFilename = bUseMask ? VRWS_MASK_FILENAME : VRWS_UNMASK_FILENAME;
std::string strRuntimeWorspacePath = VRWS_PATH + strVrwsFilename + ".vrws";

// open the given workspace
vidi_runtime_open_workspace_from_file(strVrwsFilename.c_str(), strRuntimeWorspacePath.c_str());
{% endhighlight %}

Two runtime workspaces are provided to compare the effectivess of dynamic masking.
- Resources/vidi-runtime-workspace/Advanced Tutorial 4 - Dynamical Masking (solution)_21_unmask.vrws
- Resources/vidi-runtime-workspace/Advanced Tutorial 4 - Dynamical Masking (solution)_21_mask.vrws

The runtime workspace `Advanced Tutorial 4 - Dynamical Masking (solution)_21_unmask.vrws` is generated from only GREEN tool and `Advanced Tutorial 4 - Dynamical Masking (solution)_21_mask.vrws` is generated fomr the toolchain of RED and GREEN tools that are responsible for creating dynamic mask images and classification respectively.

Depending on the flag `bUseMask` the runtime workspace to be used is chosen between two runtime workspaces.

<br/>
#### Get Image File List
{% highlight c++ %}
// get image file list
std::vector<std::string> vecTestImageList = get_all_files_names_within_folder(TEST_IMG_PATH);
{% endhighlight %}

`get_all_files_names_within_folder()` function is called to fetch the list of test image files located in `Resources/test-images`.

{% highlight c++ %}
/*
*	@function : get_all_files_names_within_folder
*   @description : get file list within folder
*/
std::vector<std::string> get_all_files_names_within_folder(std::string folder)
{
	std::vector<std::string> names;
	std::string search_path = folder + "/*.*";
	WIN32_FIND_DATA fd;
	HANDLE hFind = ::FindFirstFile(search_path.c_str(), &fd);

	if (hFind != INVALID_HANDLE_VALUE) {
		do {
			// read all (real) files in current folder
			// , delete '!' read other 2 default folder . and ..
			if (!(fd.dwFileAttributes & FILE_ATTRIBUTE_DIRECTORY)) {
				names.push_back(fd.cFileName);
			}
		} while (::FindNextFile(hFind, &fd));
		::FindClose(hFind);
	}
	return names;
}
{% endhighlight %}

Five pair of test & mask images are provided in this example. Each test image is coupled with mask image whose prefix is `mask-`. For example,
* Test Image : `Cat12-00007.png`
* Corresponding Mask Image : `mask-Cat12-00007.png`

<br/>
#### Load Test Image
{% highlight c++ %}
VIDI_IMAGE image;

// init test image
vidi_init_image(&image);

// load the current test image
vidi_load_image(strCurrentImagePath.c_str(), &image);
{% endhighlight %}

`VIDI_IMAGE` is nothing but a container of image information and is of `struct` type.
After initializing image using `vidi_init_image()` function (actually, getting the memory resource for `VIDI_IMAGE`) image is loaded from the current image being processed using `vidi_load_image()` function.

<br/>
#### Initialize Sample and Add Image
{% highlight c++ %}
// as of ViDi Suite 3.0.0, samples are processed in a few steps instead of just calling vidi_runtime_process
// the first step is to initialize the sample
vidi_runtime_create_sample(strVrwsFilename.c_str(), STREAM_NAME, SAMPLE_NAME);

// then add the image to be processed
vidi_runtime_sample_add_image(strVrwsFilename.c_str(), STREAM_NAME, SAMPLE_NAME, &image);
{% endhighlight %}

We create a runtime sample using `vidi_runtime_create_sample()` with `SAMPLE_NAME` defined as a global constant.
Then the function `vidi_runtime_sample_add_image()` is called to add the current image being processed.

<br/>
#### Load Mask Image
{% highlight c++ %}
VIDI_IMAGE maskImage;

// init mask image
vidi_init_image(&maskImage);

// load the current mask image file
vidi_load_image(strCurrentMaskImagePath.c_str(), &maskImage);
{% endhighlight %}

If mask mode is on (`bUseMask == true`), load a mask image (same procedure as loading test image).

<br/>
#### Add Mask Image as ROI into View
{% highlight c++ %}
// set masking
vidi_runtime_roi_set_mask(strVrwsFilename.c_str(), STREAM_NAME, TOOL_NAME, &maskImage);

// set ROI Manual to external
vidi_runtime_tool_set_parameter(strVrwsFilename.c_str(), STREAM_NAME, TOOL_NAME, "roi/manual/external", "true");

// add ROI view
vidi_runtime_roi_add_view(
    strVrwsFilename.c_str(),
    STREAM_NAME,
    TOOL_NAME,
    SAMPLE_NAME,
    0,
    0,
    1.0,
    0.0,
    0.0,
    1.0,
    0.0,
    0.0,
    &maskImage);
{% endhighlight %}

Now we set the ROI with the mask image using the functio `vidi_runtime_roi_set_mask()`.
After setting ROI manual to external using the function `vidi_runtime_tool_set_parameter()` add the ROI into the current view.
Regarding the parameters of the function `vidi_runtime_roi_add_view()` please refer to the header `vidi_runtime.h` for more information.

<br/>
#### Process Sample and Get Results
{% highlight c++ %}
vidi_runtime_sample_process(strVrwsFilename.c_str(), STREAM_NAME, TOOL_NAME, SAMPLE_NAME, "");

// the next step is to get the results
vidi_runtime_get_sample(strVrwsFilename.c_str(), STREAM_NAME, SAMPLE_NAME, &result_buffer);
{% endhighlight %}

Now all things are prepared for processing of runtime dynamic masking.
After processing the sample we get the processed results from `result_buffer` using the function `vidi_runtime_get_sample()`.

<br/>
#### Save Results
After the sample is processed the result information is contained in `result_buffer` in XML structure.
We have two processing outputs - XML data and overlay graphics image.

XML file is saved with output file stream `std::ofstream` from C++ standard library.

{% highlight c++ %}
std::ofstream ofs(strResultFilePath);
ofs << result_buffer.data;
{% endhighlight %}

On opening a XML file saved we can the tag structure that looks like:

{% highlight xml %}
<sample name='my_sample'>
	<tool id='Classify' type='green' uuid='83f4736e-d124-4b1a-9db5-b8aaaf11d24b'>
	</tool>
	<image>
		<marking tool_type='green' tool_uuid='83f4736e-d124-4b1a-9db5-b8aaaf11d24b' processed='2018-Nov-08 21:01:32.720911' duration='0.0073981'>
			<image_info size='750x500' channels='1' depth='8'/>
			<view size='750x500' ref_pose='1.000000,0.000000,0.000000,1.000000,0.000000,0.000000' pose='1.000000,0.000000,0.000000,1.000000,0.000000,0.000000' group=''>
				<resource name='mask' type='image' uuid='3c103561-b82c-4994-8f37-73d2579c5cd6'/>
				<green threshold='0.000000' exclusive='true'>
				<tag name='Cat12' score='1.000000'/>
			</green>
		</view>
	</marking>
</image>
</sample>
{% endhighlight %}

In order to print out some information we want it is needed to be parsed and I provide a function `PrintResults()` to get parsed `Score` and `Tags` predicted from ViDi runtime workspace.

{% highlight c++ %}
/*
*	@function : PrintResults
*   @description : print results from result buffer
*/
void PrintResults(VIDI_BUFFER result_buffer)
{
	if (result_buffer.size == 0)
	{
		std::clog << "ERROR : No result buffer" << std::endl;
		return;
	}

	// parse result_buffer to get data
	rapidxml::xml_document<> doc;
	rapidxml::xml_node<> *root_node = NULL;
	rapidxml::xml_node<> *tool_node = NULL;
	rapidxml::xml_node<> *image_node = NULL;
	rapidxml::xml_node<> *marking_node = NULL;
	rapidxml::xml_node<> *view_node = NULL;
	rapidxml::xml_node<> *tool_type_node = NULL;

	doc.parse<0>(result_buffer.data);

	try { root_node = doc.first_node("sample"); }
	catch (const std::exception&) { return; }

	try { tool_node = root_node->first_node("tool"); }
	catch (const std::exception&) { return; }

	try { tool_node = root_node->first_node("tool"); }
	catch (const std::exception&) { return; }

	try { image_node = root_node->first_node("image"); }
	catch (const std::exception&) { return; }

	try { marking_node = image_node->first_node("marking"); }
	catch (const std::exception&) { return; }

	try { view_node = marking_node->first_node("view"); }
	catch (const std::exception&) { return; }

	try { tool_type_node = view_node->first_node(tool_node->first_attribute("type")->value()); }
	catch (const std::exception&) { return; }

	int nCnt = 0;
	for (rapidxml::xml_node<> *tag_node = tool_type_node->first_node("tag"); tag_node; tag_node = tag_node->next_sibling())
	{
		std::string strTagName = tag_node->first_attribute("name")->value();
		std::string strScore = tag_node->first_attribute("score")->value();

		nCnt++;
		std::clog << "Tag-" << nCnt << " : " << strTagName << ", Score : " << strScore << std::endl;
	}
}
{% endhighlight %}

<br/>
#### Get Overlay Graphics Image and Save It
{% highlight c++ %}
// create an image to hold an overlay
VIDI_IMAGE overlay;

vidi_init_image(&overlay);

// retrieve the result overlay image
vidi_runtime_get_overlay(strVrwsFilename.c_str(), STREAM_NAME, TOOL_NAME, SAMPLE_NAME, -1, "", &overlay);

vidi_save_image(strResultFilePath.c_str(), &overlay);
vidi_free_image(&overlay);
{% endhighlight %}

Overlay grphics image is an image that only contains processing results - score for Red/Green tool, detected markings for Red too, etc.

<br/>
#### Blend Images
In masking mode being on there are 3 images to be blended - original test image, mask image and overlay graphics image.

<img src="{{ site.baseurl }}/assets/posts/2018-11-07-ViDiCppRuntimeDynamicMasking/02.png">

Regarding alph blending and image overlay please refer to the functions `AlphaBlending()` and `OverlayImage()` described in the whole code part.

------
## Verification
Verification was done by comparing between the results from ViDi production mode and those from C++ API Runtime code.

The results are taken from the previous post - [C# Runtime Dynamic Masking](https://cognexkorea.github.io/deep-learning/2018/10/29/ViDiCSharpRuntimeDynamicMasking.html).

<img src="{{ site.baseurl }}/assets/posts/2018-10-30-ViDiCSharpRuntimeDynamicMasking/04.png">

The first trial is test without dynamic masking and the result display is shown as (Note that `Tag-1` is the best tag):

<img src="{{ site.baseurl }}/assets/posts/2018-11-07-ViDiCppRuntimeDynamicMasking/03.png">

As you can see, we get the same results from C# and C++ API (Note that Scores are displayed not in percetage).

Also for the test with dynamic masking as our second trial the result window shown:

<img src="{{ site.baseurl }}/assets/posts/2018-11-07-ViDiCppRuntimeDynamicMasking/04.png">

Five images are randomly chosen among mismatches to compare with and without masks for both runtime code and production mode in ViDi.

The mask images are exported manually for the corresponding images (Edit Mask > Export).

<img src="{{ site.baseurl }}/assets/posts/2018-10-30-ViDiCSharpRuntimeDynamicMasking/05.png">

The name of each file contains its category. For example the file `Cat12-00007.png` belongs to `Cat12` class.

As we can see from the table above, if we round off the numbers of `Dynamic Masking using Runtime API` to the nearest hundredths the same results between two cases, which means dynamic masking turned out to be proven.

------
## Entire Code
I'd finish this blog post whih the entire code:

{% highlight c++ %}
/*
// ViDi.Cpp.Runtime.Dynamic.Masking.cpp : Defines the entry point for the console application.

 @ author: Alex Choi
 @ date: Nov.08, 2018
 @ contact: alex.choi@cognex.com
 @ description: This is a runtime dynamic masking example using ViDi C API.
                You can use this source in any form you want.

*/

#include "vidi.h"
#include "vidi_runtime.h"

#include <stdio.h>
#include <tchar.h>
#include <string>
#include <iostream>
#include <fstream>
#include <vector>
#include <Windows.h>
#include <filesystem>

#include "opencv2\opencv.hpp"

// RapidXML is a lightweight, header only, fast xml parser.
#define USE_RAPIDXML
#ifdef USE_RAPIDXML
#include "rapidxml.hpp"
#endif

#define CHECK_DLL_CALL(f) if(int status = f != 0) {std::cerr << get_last_error_message(status) << std::endl; vidi_deinitialize(); return -1;}

////////////////////////////////////////////////////////////
// DEFINE GLOBAL VARIABLES
////////////////////////////////////////////////////////////
const std::string VRWS_PATH("../Resources/vidi-runtime-workspace/");
const std::string VRWS_MASK_FILENAME("Advanced Tutorial 4 - Dynamical Masking (solution)_21_mask");
const std::string VRWS_UNMASK_FILENAME("Advanced Tutorial 4 - Dynamical Masking (solution)_21_unmask");

const std::string RESULT_PATH = "../Results/";

const char* STREAM_NAME = "default";
const char* TOOL_NAME = "Classify";
const char* SAMPLE_NAME = "my_sample";

const std::string TEST_IMG_PATH("../Resources/test-images");
const std::string MASK_IMG_PATH("../Resources/mask-images");

// flag for using dynamic mask
const bool bUseMask = true;


/*
*	@function : get_last_error_message
*/
std::string get_last_error_message(int status)
{
	VIDI_BUFFER buffer;
	vidi_init_buffer(&buffer);

	VIDI_UINT internal_status;

	internal_status = vidi_get_error_message(status, &buffer);
	if (internal_status != VIDI_SUCCESS)
		return "failed to get last error message";

#ifdef USE_RAPIDXML
	rapidxml::xml_document<> doc;
	doc.parse<0>(buffer.data);

	std::string error_message(doc.first_node("error")->value());
#else
	std::string error_message = buffer.data;
#endif

	return error_message;
}


/*
*	@function : pathAppend
*   @description : combine paths
*/
std::string pathAppend(const std::string& p1, const std::string& p2)
{
	char sep = '/';
	std::string tmp = p1;

#ifdef _WIN32
	sep = '\\';
#endif

	if (p1[p1.length()] != sep)
	{ // Need to add a path separator
		tmp += sep;
		return(tmp + p2);
	}
	else
		return(p1 + p2);
}


/*
*	@function : get_all_files_names_within_folder
*   @description : get file list within folder
*/
std::vector<std::string> get_all_files_names_within_folder(std::string folder)
{
	std::vector<std::string> names;
	std::string search_path = folder + "/*.*";
	WIN32_FIND_DATA fd;
	HANDLE hFind = ::FindFirstFile(search_path.c_str(), &fd);

	if (hFind != INVALID_HANDLE_VALUE) {
		do {
			// read all (real) files in current folder
			// , delete '!' read other 2 default folder . and ..
			if (!(fd.dwFileAttributes & FILE_ATTRIBUTE_DIRECTORY)) {
				names.push_back(fd.cFileName);
			}
		} while (::FindNextFile(hFind, &fd));
		::FindClose(hFind);
	}
	return names;
}


/*
*	@function : CreateFolder
*   @description : if folder not exists, create it
*/
void CreateFolder(const char * path)
{
	if (!CreateDirectory(path, NULL))
	{
		return;
	}
}


/*
*	@function : StringTokenize
*   @description : string tokenization
*/
std::vector<std::string> StringTokenize(const char *str, char c = ' ')
{
	std::vector<std::string> result;

	do {
		const char *begin = str;

		while (*str != c && *str)
			str++;

		result.push_back(std::string(begin, str));
	} while (0 != *str++);

	return result;
}


/*
*	@function : PrintResults
*   @description : print results from result buffer
*/
void PrintResults(VIDI_BUFFER result_buffer)
{
	if (result_buffer.size == 0)
	{
		std::clog << "ERROR : No result buffer" << std::endl;
		return;
	}

	// parse result_buffer to get data
	rapidxml::xml_document<> doc;
	rapidxml::xml_node<> *root_node = NULL;
	rapidxml::xml_node<> *tool_node = NULL;
	rapidxml::xml_node<> *image_node = NULL;
	rapidxml::xml_node<> *marking_node = NULL;
	rapidxml::xml_node<> *view_node = NULL;
	rapidxml::xml_node<> *tool_type_node = NULL;

	doc.parse<0>(result_buffer.data);

	try { root_node = doc.first_node("sample"); }
	catch (const std::exception&) { return; }

	try { tool_node = root_node->first_node("tool"); }
	catch (const std::exception&) { return; }

	try { tool_node = root_node->first_node("tool"); }
	catch (const std::exception&) { return; }

	try { image_node = root_node->first_node("image"); }
	catch (const std::exception&) { return; }

	try { marking_node = image_node->first_node("marking"); }
	catch (const std::exception&) { return; }

	try { view_node = marking_node->first_node("view"); }
	catch (const std::exception&) { return; }

	try { tool_type_node = view_node->first_node(tool_node->first_attribute("type")->value()); }
	catch (const std::exception&) { return; }

	int nCnt = 0;
	for (rapidxml::xml_node<> *tag_node = tool_type_node->first_node("tag"); tag_node; tag_node = tag_node->next_sibling())
	{
		std::string strTagName = tag_node->first_attribute("name")->value();
		std::string strScore = tag_node->first_attribute("score")->value();

		nCnt++;
		std::clog << "Tag-" << nCnt << " : " << strTagName << ", Score : " << strScore << std::endl;
	}
}


/*
*	@function : PrintResults
*   @description : print results from buffer
*/
cv::Mat AlphaBlending(cv::Mat background, cv::Mat foreground, cv::Mat mask)
{
	// Convert Mat to float data type
	foreground.convertTo(foreground, CV_32FC3);
	background.convertTo(background, CV_32FC3);

	// Normalize the alpha mask to keep intensity between 0 and 1
	mask.convertTo(mask, CV_32FC3, 1.0 / 255);

	// Storage for output image
	cv::Mat outImage = cv::Mat::zeros(foreground.size(), foreground.type());

	// Multiply the foreground with the alpha matte
	multiply(mask, foreground, foreground);

	// Multiply the background with ( 1 - alpha )
	multiply(cv::Scalar::all(1.0) - mask, background, background);

	// Add the masked foreground and background.
	add(foreground, background, outImage);

	return outImage;
}


/*
*	@function : OverlayImage
*   @description : Put ViDi result image(.png format with transparency channel) on top of the original one
*/
void OverlayImage(cv::Mat* src, cv::Mat* overlay)
{
	for (int y = 0; y < src->rows; ++y)
	{
		int fY = y;

		if (fY >= overlay->rows)
			break;

		for (int x = 0; x < src->cols; ++x)
		{
			int fX = x;

			if (fX >= overlay->cols)
				break;

			double opacity = ((double)overlay->data[fY * overlay->step + fX * overlay->channels() + 3]) / 255;

			for (int c = 0; opacity > 0 && c < src->channels(); ++c)
			{
				unsigned char overlayPx = overlay->data[fY * overlay->step + fX * overlay->channels() + c];
				unsigned char srcPx = src->data[y * src->step + x * src->channels() + c];
				src->data[y * src->step + src->channels() * x + c] = (uchar)(srcPx * (1. - opacity) + overlayPx * opacity);
			}
		}
	}
}


/*
*	@function : main
*/
int main()
{
	// send debug info to a message file
	VIDI_UINT status = vidi_debug_infos(VIDI_DEBUG_SINK_FILE, "vidi_messages.log");
	if (status != VIDI_SUCCESS)
	{
		std::cerr << "failed to enable debug infos" << std::endl;
		return -1;
	}

	// initialize the libary to run with one GPU per tool
	status = vidi_initialize(VIDI_GPU_SINGLE_DEVICE_PER_TOOL, "");
	if (status != VIDI_SUCCESS)
	{
		std::clog << "failed to initialize library" << std::endl;
		return -1;
	}

	// create and initialize a buffer to be used whenever data is returned from the library
	VIDI_BUFFER buffer;
	status = vidi_init_buffer(&buffer);
	if (status != VIDI_SUCCESS)
	{
		std::clog << "failed to initialize vidi buffer" << std::endl;
		return -1;
	}

	// define runtime workspace path
	std::string strVrwsFilename = bUseMask ? VRWS_MASK_FILENAME : VRWS_UNMASK_FILENAME;
	std::string strRuntimeWorspacePath = VRWS_PATH + strVrwsFilename + ".vrws";

	// open the given workspace
	status = vidi_runtime_open_workspace_from_file(strVrwsFilename.c_str(), strRuntimeWorspacePath.c_str());
	if (status != VIDI_SUCCESS)
	{
		vidi_get_error_message(status, &buffer);

		// if you get this error, it's possible that the resources were not extracted in the path
		std::clog << "failed to open workspace: " << buffer.data << std::endl;
		vidi_deinitialize();
		return -1;
	}

	// get image file list
	std::vector<std::string> vecTestImageList = get_all_files_names_within_folder(TEST_IMG_PATH);

	/////////////////////////////////////////////////////////////////////////////////////////////
	// process image one after another
	/////////////////////////////////////////////////////////////////////////////////////////////
	for (int i = 0; i < vecTestImageList.size(); i++)
	{
		VIDI_IMAGE image;

		// init test image
		status = vidi_init_image(&image);
		if (status != VIDI_SUCCESS)
		{
			vidi_get_error_message(status, &buffer);
			std::clog << "failed to initialize image: " << buffer.data << std::endl;
			vidi_deinitialize();
			return -1;
		}

		// load the current test image
		std::string strCurrentImageFilename = vecTestImageList[i];
		std::string strCurrentImagePath = pathAppend(TEST_IMG_PATH, strCurrentImageFilename);
		status = vidi_load_image(strCurrentImagePath.c_str(), &image);
		if (status != VIDI_SUCCESS)
		{
			vidi_get_error_message(status, &buffer);

			// if you get this error, it's possible that the resources were not extracted in the path
			std::clog << "failed to read image: " << buffer.data << std::endl;
			vidi_deinitialize();
			return -1;
		}
		else
		{
			std::clog << std::endl;
			std::clog << "====================================================================================" << std::endl;
			std::clog << "Current image name being processed : " << strCurrentImageFilename << std::endl;
			std::clog << "====================================================================================" << std::endl;
			std::clog << std::endl;
		}

		// init result buffer
		VIDI_BUFFER result_buffer;
		status = vidi_init_buffer(&result_buffer);
		if (status != VIDI_SUCCESS)
		{
			std::clog << "failed to initialize result buffer" << std::endl;
			vidi_deinitialize();
			return -1;
		}

		// as of ViDi Suite 3.0.0, samples are processed in a few steps instead of just calling vidi_runtime_process
		// the first step is to initialize the sample
		status = vidi_runtime_create_sample(strVrwsFilename.c_str(), STREAM_NAME, SAMPLE_NAME);
		if (status != VIDI_SUCCESS)
		{
			std::clog << "failed to initialize sample" << std::endl;
			vidi_deinitialize();
			return -1;
		}

		// then add the image to be processed
		status = vidi_runtime_sample_add_image(strVrwsFilename.c_str(), STREAM_NAME, SAMPLE_NAME, &image);
		if (status != VIDI_SUCCESS)
		{
			std::clog << "failed to add image" << std::endl;
			vidi_deinitialize();
			return -1;
		}

		std::string strCurrentMaskImagePath = pathAppend(MASK_IMG_PATH, "mask-" + vecTestImageList[i]);

		if (bUseMask) // if dynamic masking mode used
		{
			VIDI_IMAGE maskImage;

			// init mask image
			status = vidi_init_image(&maskImage);
			if (status != VIDI_SUCCESS)
			{
				vidi_get_error_message(status, &buffer);
				std::clog << "failed to initialize image: " << buffer.data << std::endl;
				vidi_deinitialize();
				return -1;
			}

			// load the current mask image file
			status = vidi_load_image(strCurrentMaskImagePath.c_str(), &maskImage);
			if (status != VIDI_SUCCESS)
			{
				vidi_get_error_message(status, &buffer);

				// if you get this error, it's possible that the resources were not extracted in the path
				std::clog << "failed to read mask image: " << buffer.data << std::endl;
				vidi_deinitialize();
				return -1;
			}

			// set masking
			status = vidi_runtime_roi_set_mask(strVrwsFilename.c_str(), STREAM_NAME, TOOL_NAME, &maskImage);
			if (status != VIDI_SUCCESS)
			{
				vidi_get_error_message(status, &buffer);

				// if you get this error, it's possible that the resources were not extracted in the path
				std::clog << "failed to set mask image: " << buffer.data << std::endl;
				vidi_deinitialize();
				return -1;
			}

			// set ROI Manual to external
			status = vidi_runtime_tool_set_parameter(strVrwsFilename.c_str(), STREAM_NAME, TOOL_NAME, "roi/manual/external", "true");
			if (status != VIDI_SUCCESS)
			{
				vidi_get_error_message(status, &buffer);

				// if you get this error, it's possible that the resources were not extracted in the path
				std::clog << "failed to set mask image: " << buffer.data << std::endl;
				vidi_deinitialize();
				return -1;
			}

			// add ROI view
			status = vidi_runtime_roi_add_view(
				strVrwsFilename.c_str(),
				STREAM_NAME,
				TOOL_NAME,
				SAMPLE_NAME,
				0,
				0,
				1.0,
				0.0,
				0.0,
				1.0,
				0.0,
				0.0,
				&maskImage);
			if (status != VIDI_SUCCESS)
			{
				vidi_get_error_message(status, &buffer);

				// if you get this error, it's possible that the resources were not extracted in the path
				std::clog << "failed to add view: " << buffer.data << std::endl;
				vidi_deinitialize();
				return -1;
			}

			vidi_free_image(&maskImage);
		}

		//
		/**
		* subsequently process the sample for each tool
		* here, we know that the tool that's being processed is called "analyze". If you do not know the name of the
		* tool or tools that are being processed, you can use vidi_runtime_list_tools(). Since there is only one tool,
		* this is called only once. You can also trigger the whole chain to process by calling
		* vidi_runtime_process_sample for the last tool in the chain.
		*/
		status = vidi_runtime_sample_process(strVrwsFilename.c_str(), STREAM_NAME, TOOL_NAME, SAMPLE_NAME, "");
		if (status != VIDI_SUCCESS)
		{
			vidi_get_error_message(status, &buffer);
			std::clog << "failed to process sample: " << buffer.data << std::endl;
			vidi_deinitialize();
			return -1;
		}

		// the next step is to get the results
		status = vidi_runtime_get_sample(strVrwsFilename.c_str(), STREAM_NAME, SAMPLE_NAME, &result_buffer);
		if (status != VIDI_SUCCESS)
		{
			std::clog << "failed to get results from sample" << std::endl;
			vidi_deinitialize();
			return -1;
		}

		// save result_buffer as xml file
		std::vector<std::string> vecTemp = StringTokenize(vecTestImageList[i].c_str(), '.');
		std::string strResultFilePath = vecTemp[vecTemp.size() - 2] + ".xml";
		strResultFilePath = pathAppend(RESULT_PATH, strResultFilePath);
		std::ofstream ofs(strResultFilePath);
		ofs << result_buffer.data;

		// parse result_buffer to get data
		PrintResults(result_buffer);

		std::clog << std::endl;
		std::clog << "writing result to " << strResultFilePath << std::endl;

		// deallocate memory
		vidi_free_buffer(&result_buffer);
		vidi_free_image(&image);

		// below shows how to get and save an overlay image from a tool
		{
			// create an image to hold an overlay
			VIDI_IMAGE overlay;

			status = vidi_init_image(&overlay);
			if (status != VIDI_SUCCESS)
			{
				std::clog << "failed to initialize overlay image" << std::endl;
				vidi_deinitialize();
				return -1;
			}

			// retrieve the result overlay image
			status = vidi_runtime_get_overlay(strVrwsFilename.c_str(), STREAM_NAME, TOOL_NAME, SAMPLE_NAME, -1, "", &overlay);
			if (status != VIDI_SUCCESS)
			{
				vidi_get_error_message(status, &buffer);
				std::clog << "failed to retrieve overlay image: " << buffer.data << std::endl;
				vidi_deinitialize();
				return -1;
			}

			strResultFilePath = "OVERLAY-" + vecTemp[vecTemp.size() - 2] + ".png";
			strResultFilePath = pathAppend(RESULT_PATH, strResultFilePath);

			std::clog << "writing result overlay to " << strResultFilePath << std::endl;
			vidi_save_image(strResultFilePath.c_str(), &overlay);
			vidi_free_image(&overlay);

			// now that we've gotten the overlay, we can free the sample
			status = vidi_runtime_free_sample(strVrwsFilename.c_str(), STREAM_NAME, SAMPLE_NAME);
			if (status != VIDI_SUCCESS)
			{
				std::clog << "failed to free sample" << std::endl;
				vidi_deinitialize();
				return -1;
			}

			// Read the images
			cv::Mat background; // original image
			cv::Mat foreground; // overlay graphics image
			cv::Mat mask = cv::imread(strCurrentMaskImagePath);; // mask image

			// image blending (original + mask + overlay)
			if (bUseMask)
			{
				// Read the images
				background = cv::imread(strCurrentImagePath);
				foreground = AlphaBlending(background, cv::imread(strResultFilePath), mask);
			}
			else
			{
				// Read the images
				background = cv::imread(strCurrentImagePath, cv::IMREAD_UNCHANGED);
				foreground = cv::imread(strResultFilePath, cv::IMREAD_UNCHANGED);
				OverlayImage(&foreground, &background);
			}

			// save image
			std::string strOutputImagePath = pathAppend(RESULT_PATH, "COMBINED-" + strCurrentImageFilename);
			cv::imwrite(strOutputImagePath, foreground);
		}
	}

	// this will free all images and buffers which have not yet been freed

	vidi_deinitialize();

	std::cout << std::endl;
	std::cout << "Press any key to exit...." << std::endl;
	std::cin.get();

    return 0;
}
{% endhighlight %}
