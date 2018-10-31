---
layout: post
title:  "ViDi API / C++ Runtime API Example"
date:   2018-07-03 11:07:00
author: Alex Choi
categories: Deep-Learning
---

## 개발 및 실행환경
- OS : Windows 10 x64
- IDE : MS Visual Studio 2015 (v14)
- ViDi : v3.1
- 개발언어: C++

본 샘플 프로그램 테스트를 위해서는 다음 두 가지가 준비되어 있어야 합니다:

- ViDi Runtime Workspace: .vrws 확장자명을 갖는 ViDi Runtime Workspace가 있어야 하며, 없는 경우 ViDi GUI Suite로부터 Runtime Workspace를 Export합니다.
- 테스트 이미지

------
## 개발환경 구축
ViDi 3.1이 Default Option으로 설치된 경우, ViDi API 관련 header 및 library 파일은 아래와 같은 경로에 설치됩니다:

[vidi_31.lib 경로]
{% highlight text %}
C:\Program Files\Cognex\ViDi Suite\3.1
{% endhighlight %}

[header 파일 경로]
{% highlight text %}
C:\Program Files\Cognex\ViDi Suite\3.1\develop\include
{% endhighlight %}

[Third Party 라이브러리 경로]
{% highlight text %}
C:\ProgramData\Cognex\ViDi Suite\3.1\Examples\include
{% endhighlight %}

[ViDi API 예제 경로]
{% highlight text %}
C:\ProgramData\Cognex\ViDi Suite\3.1\Examples
{% endhighlight %}

<br/>
#### 추가 Header File 경로 지정
Visual Studio에서 프로젝트의 `속성 페이지(Property Pages) > C/C++ > General > Additional Include Directories(추가 포함 디렉터리)`에 ViDi Header Files 경로를 추가합니다:

{% highlight text %}
$(VIDI_BIN_V31)develop/include
{% endhighlight %}

매크로 상수 `$(VIDI_BIN_V31)`는 ViDi 설치 후 등록되며 `C:\Program Files\Cognex\ViDi Suite\3.1\` 경로를 지정합니다.

<img src="{{ site.baseurl }}/assets/posts/2018-07-03-CppRuntimeApi/01.png">

추가적으로 [rapidxml](http://rapidxml.sourceforge.net/) 라이브러리의 Header 파일을 추가합니다. rapidxml은, ViDi Runtime 프로세싱의 결과를 XML 형식으로 저장되는데 이를 처리하기 위해 필요한 Third Party 라이브러리입니다.

Visual Studio에서 프로젝트의 `속성 페이지(Property Pages) > C/C++ > General > Additional Include Directories(추가 포함 디렉터리)`에 ViDi Header Files 경로를 추가합니다:

{% highlight text %}
C:\ProgramData\Cognex\ViDi Suite\3.1\Examples\include
{% endhighlight %}

<img src="{{ site.baseurl }}/assets/posts/2018-07-03-CppRuntimeApi/02.png">

<br/>
#### 추가 라이브러리 경로 지정
Visual Studio에서 프로젝트의 `속성 페이지(Property Pages) > Linker > General > Additional Library Directories`에 `$(VIDI_BIN_V31)`을 추가합니다.

<img src="{{ site.baseurl }}/assets/posts/2018-07-03-CppRuntimeApi/03.png">

<br/>
#### 추가 종속성 지정
Visual Studio에서 프로젝트의 `속성 페이지(Property Pages) > Linker > Input > Additional Dependencies`에 `vidi_31.lib`을 추가합니다.

<img src="{{ site.baseurl }}/assets/posts/2018-07-03-CppRuntimeApi/04.png">

------
## 샘플코드 설명
본 샘플코드는 지정된 샘플 이미지를 처리하도록 되어 있으며, 카메라로부터 획득된 이미지 버퍼를 처리하거나 이미지 폴더 내의 이미지들을 일괄적으로 처리하려면 별도의 코드 작성이 필요함을 알려 드립니다.

아래 다운로드를 링크를 클릭하여 Visual Studio 샘플 솔루션을 다운로드 하세요.

[Visual Studio 솔루션 파일 다운로드](http://13.125.95.237/SDI/SDI.SealPin.zip)

<br/>
#### 매크로 상수 정의
본 샘플에서 ViDi Workspace 및 테스트 이미지 경로 등을 다음과 같이 매크로로 정의하였습니다.

{% highlight text %}
// macro
#define WORKSPACE_NAME		"SDI_SealPin"
#define WORKSPACE_PATH		"D:\\Projects\\COGNEX\\SDI-SealPin\\ViDI"
#define TEST_IMAGE_PATH		"..\\test_images\\20180504152541_36011842920173IO8P1B_L1.bmp"
#define STREAM_NAME		"default"
#define SAMPLE_NAME		"my_sample"
#define TOOL_NAME		"Analyze"
#define LOG_PATH		"..\\log"
#define RESULT_PATH		"..\\results"
{% endhighlight %}

|   매크로 상수   |                   설명                   |
|:---------------:|:----------------------------------------:|
|  <font face="Consolas">WORKSPACE_NAME</font> | <font face="Consolas">ViDi Workspace</font> 파일명 정의               |
|  <font face="Consolas">WORKSPACE_PATH</font> | <font face="Consolas">ViDi Workspace</font> 파일 경로 정의            |
| <font face="Consolas">TEST_IMAGE_PATH</font> | <font face="Consolas">Test Image</font> 파일 경로 정의                |
|   <font face="Consolas">STREAM_NAME</font>   | <font face="Consolas">ViDi Workspace</font> 내 Stream 이름 지정       |
|   <font face="Consolas">SAMPLE_NAME</font>   | <font face="Consolas">Sample</font> 이름 지정                         |
|    <font face="Consolas">TOOL_NAME</font>    | <font face="Consolas">ViDi Worskapce</font> 내 Tool 이름 지정         |
|     <font face="Consolas">LOG_PATH</font>    | Runtime 로그 파일 저장 경로 지정         |
| <font face="Consolas">RESULT_PATH</font>     | Runtime 실행 후 결과 파일 저장 경로 지정 |

<br/>
#### 폴더 생성
아래의 `CreateFolder()` 함수는 Runtime 로그 파일 및 Runtime 결과 파일을 저장하기 위한 폴더를 생성하기 위해 작성된 것이며, 해당 폴더가 있는지 검사 후 없는 경우 생성합니다.

{% highlight csharp %}
void CreateFolder(const char * path)
{
	if (!CreateDirectory(path, NULL))
	{
		return;
	}
}
{% endhighlight %}

`main()` 함수에서 다음과 같이 호출하여 Runtime 로그 파일 및 Runtime 결과 파일을 저장할 폴더를 생성합니다.

{% highlight csharp %}
// send debug info to a message file
std::stringstream ss1;
ss1 << LOG_PATH << "\\" << "vidi_messages.log";
std::string log_file_path = ss1.str();
VIDI_UINT status = vidi_debug_infos(VIDI_DEBUG_SINK_FILE, log_file_path.c_str());
if (status != VIDI_SUCCESS)
{
    std::cerr << "failed to enable debug infos" << endl;
    return -1;
}
{% endhighlight %}

해당 로그 파일은 `{SolutionDir}/log` 경로 내에 `vidi_message.log`라는 파일 이름으로 저장됩니다.

<br/>
#### GPU 모드 설정
사용하는 GPU 하드웨어 환경에 따라 Single GPU 또는 Multiple GPU를 설정할 수 있습니다. 아래 코드는 Single GPU 모드로 초기화한 것입니다:

{% highlight csharp %}
// initialize the libary to run with one GPU per tool
status = vidi_initialize(VIDI_GPU_SINGLE_DEVICE_PER_TOOL, "");
if (status != VIDI_SUCCESS)
{
    clog << "failed to initialize library" << endl;
    return -1;
}
{% endhighlight %}

만약 Multi GPU가 가능한 환경인 경우, `vidi_initialize()` 함수의 `VIDI_GPU_SINGLE_DEVICE_PER_TOOL`를 `VIDI_GPU_MULTIPLE_DEVICES_PER_TOOL`로 변경하시기 바랍니다.

<br/>
#### Buffer 초기화
라이브러리로부터 반환되는 데이터 값을 저장하는 buffer를 생성하고 초기화 합니다.

{% highlight csharp %}
// create and initialize a buffer to be used whenever data is returned from the library
VIDI_BUFFER buffer;
status = vidi_init_buffer(&buffer);
if (status != VIDI_SUCCESS)
{
    clog << "failed to initialize vidi buffer" << endl;
    return -1;
}
{% endhighlight %}

<br/>
#### ViDi Workspace 열기
ViDi Runtime Workspace 파일이 지정된 위치로부터 해당 파일을 엽니다:

{% highlight csharp %}
// workspace name & path
std::stringstream ss2;
std::string strWorkspaceName = WORKSPACE_NAME;
std::string strWorkspacePath = "";

ss2 << WORKSPACE_PATH << "\\" << WORKSPACE_NAME << ".vrws";
strWorkspacePath = ss2.str();

// open the given workspace
status = vidi_runtime_open_workspace_from_file(strWorkspaceName.c_str(), strWorkspacePath.c_str());
if (status != VIDI_SUCCESS)
{
    vidi_get_error_message(status, &buffer);
    // if you get this error, it's possible that the resources were not extracted in the path
    clog << "failed to open workspace: " << buffer.data << endl;
    vidi_deinitialize();
    return -1;
}
{% endhighlight %}

<br/>
#### 결과를 저장할 Buffer 생성
Runtime을 통해 얻어진 결과 데이터를 저장할 Buffer를 생성합니다.

{% highlight csharp %}
VIDI_BUFFER result_buffer;
status = vidi_init_buffer(&result_buffer);
if (status != VIDI_SUCCESS)
{
    clog << "failed to initialize result buffer" << endl;
    vidi_deinitialize();
    return -1;
}
{% endhighlight %}

<br/>
#### VIDI_IMAGE 초기화 및 이미지 로딩
ViDi에 `VIDI_IMAGE`라는 구조체(struct) 타입으로 이미지를 공급합니다. `VIDI_IMAGE` 구조체의 원형을 살펴보면 다음과 같습니다:

{% highlight csharp %}
/** @brief Structure holding an image */
typedef struct
{
    /**@brief width of the image*/
    VIDI_UINT 	width;
    /**@brief height of the image*/
    VIDI_UINT 	height;
    /**@brief number of channels of the image*
    *
    *	Supported number of channels 1-4
    */
    VIDI_UINT	channels;
    /**@brief image data*/
    void* 		data;
    /**@brief depth of a channel
    *  \ref VIDI_IMG_8U or \ref VIDI_IMG_16U
    */
    VIDI_UINT 	channel_depth;
    /** @brief
    *	distance in memory between 2 pixels
    *	of the same column for 2 consecutive rows.
    *
    *	Usually step is equal to width
    *
    */
    VIDI_UINT 	step;
} VIDI_IMAGE;
{% endhighlight %}

이미지 파일은 ViDi C API 함수 `vidi_load_image()`를 통해 `VIDI_IMAGE` 구조로 변경됩니다. 이미지를 초기화하고 로딩하는 코드는 다음과 같습니다:

{% highlight csharp %}
VIDI_IMAGE image;
status = vidi_init_image(&image);
if (status != VIDI_SUCCESS)
{
    vidi_get_error_message(status, &buffer);
    clog << "failed to initialize image: " << buffer.data << endl;
    vidi_deinitialize();
    return -1;
}

status = vidi_load_image(TEST_IMAGE_PATH, &image);
if (status != VIDI_SUCCESS)
{
    vidi_get_error_message(status, &buffer);
    // if you get this error, it's possible that the resources were not extracted in the path
    clog << "failed to read image: " << buffer.data << endl;
    vidi_deinitialize();
    return -1;
}
{% endhighlight %}

<br/>
#### Sample 생성 및 이미지 추가
ViDi Runtime Sample을 생성하고 생성된 Sample에 이미지를 추가하는 코드입니다:

{% highlight csharp %}
// as of ViDi Suite 3.0.0, samples are processed in a few steps instead of just calling vidi_runtime_process
// the first step is to initialize the sample
status = vidi_runtime_create_sample(WORKSPACE_NAME, STREAM_NAME, SAMPLE_NAME);
if (status != VIDI_SUCCESS)
{
    clog << "failed to initialize sample" << endl;
    vidi_deinitialize();
    return -1;
}

// then add the image to be processed
status = vidi_runtime_sample_add_image(WORKSPACE_NAME, STREAM_NAME, SAMPLE_NAME, &image);
if (status != VIDI_SUCCESS)
{
    clog << "failed to add image" << endl;
    vidi_deinitialize();
    return -1;
}
{% endhighlight %}

<br/>
#### Sample 프로세싱 및 결과 가져오기
`vidi_runtime_sample_process()` API 함수를 통해 생성된 Sample을 프로세싱하고 `vidi_runtime_get_sample()` 함수를 통해 결과를 가져옵니다.

{% highlight csharp %}
/**
* subsequently process the sample for each tool
* here, we know that the tool that's being processed is called "analyze". If you do not know the name of the
* tool or tools that are being processed, you can use vidi_runtime_list_tools(). Since there is only one tool,
* this is called only once. You can also trigger the whole chain to process by calling
* vidi_runtime_process_sample for the last tool in the chain.
*/
status = vidi_runtime_sample_process(WORKSPACE_NAME, STREAM_NAME, TOOL_NAME, SAMPLE_NAME, "");
if (status != VIDI_SUCCESS)
{
    vidi_get_error_message(status, &buffer);
    clog << "failed to process sample: " << buffer.data << endl;
    vidi_deinitialize();
    return -1;
}

// the next step is to get the results
status = vidi_runtime_get_sample(WORKSPACE_NAME, STREAM_NAME, SAMPLE_NAME, &result_buffer);
if (status != VIDI_SUCCESS)
{
    clog << "failed to get results from sample" << endl;
    vidi_deinitialize();
    return -1;
}
{% endhighlight %}

결과 데이터는 `result_buffer` 포인터 변수를 통해 저장되며 Third Party 라이브러리인 rapidxml을 통해 원하는 값을 Parsing하여 얻습니다.

`result_buffer`에 저장된 XML 결과 예시는 다음과 같습니다:

{% highlight xml %}
<sample name='my_sample'>
		<tool id='Localize' type='blue' uuid='046868ac-287f-4e42-a22a-3e53206008fa'>
			<tool id='Classify' type='green' uuid='5d8ca741-e19d-4336-8132-ca78fc74d89a'>
				<tool id='Analyze' type='red' uuid='45d77689-e65f-40f5-90e8-ce73f78a0286'>
				</tool>
			</tool>
		</tool>
		<image>
			<marking tool_type='blue' tool_uuid='046868ac-287f-4e42-a22a-3e53206008fa' processed='2018-Jul-03 13:44:35.602066' duration='0.162029'>
				<image_info size='2588x1940' channels='1' depth='8'/>
				<view size='2588x1940' ref_pose='1.000000,0.000000,0.000000,1.000000,0.000000,0.000000' pose='1.000000,0.000000,0.000000,1.000000,0.000000,0.000000' group=''>
					<blue oriented='false' scaled='false'>
						<feat id='0' loc='779.920898,744.875183' angle='0.000000' size='60.000000x60.000000' score='0.999994' covers='-1'/>
						<feat id='1' loc='1300.319824,744.193176' angle='0.000000' size='60.000000x60.000000' score='0.999648' covers='-1'/>
						<match model_id='Model 2' score='0.999821' covers='-1' node_coords='-231.736542,1.432587 231.736542,-1.432587 ' string='01'>
							<pose matrix='1.122791,0.005470,-0.005470,1.122791,1040.120361,744.534180' angle='0.004871' scale='1.122804' aspect_ratio='1.000000' shear='0.000000' pos='1040.120361,744.534180'/>
							<feat idx='0'/>
							<feat idx='1'/>
						</match>
					</blue>
				</view>
			</marking>
			<marking tool_type='red' tool_uuid='45d77689-e65f-40f5-90e8-ce73f78a0286' processed='2018-Jul-03 13:44:35.760164' duration='0.079743'>
				<image_info size='2588x1940' channels='1' depth='8'/>
				<view size='745x764' ref_pose='0.999988,0.004871,-0.004871,0.999988,1040.120361,744.534180' pose='0.999988,0.004871,-0.004871,0.999988,670.335754,698.701538' group=''>
					<resource name='mask' type='image' uuid='5479a933-9e15-487c-a5ec-3b9d3572a529'/>
					<red score='0.686521' threshold='[0.183711, 0.325100]' mode='supervised'>
						<resource name='heat_map' type='image' uuid='93a4b62d-ff7f-57ed-b0b8-a81073a900c0'/>
						<resource name='score_map:defect' type='image' scale='65535' uuid='8afc6ee5-e9ee-51c6-a527-be75051dd155'/>
						<region score='0.686521' name='defect' area='1096.4' perimeter='136.4' coverage='0.0' center='52.8,494.2'
							points='50.0,475.0 47.5,477.5 42.5,477.5 40.0,480.0 40.0,482.5 37.5,485.0 37.5,512.7 40.0,515.2 42.5,515.2 45.0,517.7 50.0,517.7 52.5,515.2 55.0,515.2 55.0,512.7 57.5,510.2 57.5,507.7 60.0,505.1 60.0,502.6 62.5,500.1 65.0,500.1 67.5,497.6 70.0,497.6 72.5,495.1 72.5,480.0 70.0,480.0 67.5,477.5 60.0,477.5 57.5,475.0 '/>
					</red>
				</view>
			</marking>
			<marking tool_type='green' tool_uuid='5d8ca741-e19d-4336-8132-ca78fc74d89a' processed='2018-Jul-03 13:44:35.677939' duration='0.075819'>
				<image_info size='2588x1940' channels='1' depth='8'/>
				<view size='745x764' ref_pose='0.999988,0.004871,-0.004871,0.999988,1040.120361,744.534180' pose='0.999988,0.004871,-0.004871,0.999988,670.335754,698.701538' group=''>
					<green threshold='0.500000' exclusive='true'>
					<tag name='2' score='0.999995'/>
				</green>
			</view>
		</marking>
	</image>
</sample>
{% endhighlight %}

<img src="{{ site.baseurl }}/assets/posts/2018-07-03-CppRuntimeApi/05.png">

위의 태그 중 `<red>` 태그(또는 노드)의 score, threshold 속성의 값들과, `<region>` 태그의 score, area, perimeter, center 속성의 값들을 얻는 방법에 대한 코드는 다음과 같습니다. 단, 코드에 자세한 설명은 생략하도록 하겠습니다.

{% highlight csharp %}
// get data from XML file
rapidxml::xml_document<> doc;
rapidxml::xml_node<> *root_node = NULL;
rapidxml::xml_node<> *image_node = NULL;
rapidxml::xml_node<> *marking_node = NULL;
rapidxml::xml_node<> *view_node = NULL;
rapidxml::xml_node<> *tool_node = NULL;
rapidxml::xml_node<> *region_node = NULL;

doc.parse<0>(result_buffer.data);
root_node = doc.first_node("sample");
image_node = root_node->first_node("image");
marking_node = image_node->first_node("marking");

while (marking_node != NULL)
{
    std::string strTool_type(marking_node->first_attribute("tool_type")->value());
    if (strTool_type == "red")
    {
        break;
    }

    marking_node = marking_node->next_sibling();
}

std::string strTmp = "";
float fTop_score = 0.0;

std::string strThreshold = "";
float fLowThreshold = 0.0;
float fHighThreshold = 0.0;

std::vector<float> vecRegionScore;
std::vector<float> vecRegionArea;
std::vector<float> vecRegionPerimeter;
std::vector<float> vecRegionCenterX;
std::vector<float> vecRegionCenterY;

if (marking_node != NULL)
{
    view_node = marking_node->first_node("view");
    tool_node = view_node->first_node("red");

    // get the top score among regions
    strTmp = tool_node->first_attribute("score")->value();
    fTop_score = std::stof(strTmp.c_str(), 0);

    // get the threshold [low, high]
    strThreshold = tool_node->first_attribute("threshold")->value();

    std::vector<string> vecTemp;

    vecTemp = StringSplit(strThreshold.c_str(), '[');

    vecTemp = StringSplit(vecTemp[1].c_str(), ',');
    fLowThreshold = ::stof(vecTemp[0].c_str(), 0);

    vecTemp = StringSplit(vecTemp[1].c_str(), ']');
    fHighThreshold = ::stof(vecTemp[0].c_str(), 0);

    vecTemp.clear();

    // get the first <region> node
    region_node = tool_node->first_node("region");

    while (region_node != NULL)
    {
        // get the score of each region
        strTmp = region_node->first_attribute("score")->value();
        vecRegionScore.push_back(std::stof(strTmp.c_str(), 0));

        // get the area of each region
        strTmp = region_node->first_attribute("area")->value();
        vecRegionArea.push_back(std::stof(strTmp.c_str(), 0));

        // get the perimeter of each region
        strTmp = region_node->first_attribute("perimeter")->value();
        vecRegionPerimeter.push_back(std::stof(strTmp.c_str(), 0));

        // get the center of each region
        strTmp = region_node->first_attribute("center")->value();
        vecTemp = StringSplit(strTmp.c_str(), ',');
        vecRegionCenterX.push_back(std::stof(vecTemp[0].c_str(), 0));
        vecRegionCenterY.push_back(std::stof(vecTemp[1].c_str(), 0));

        cout << "Region Score : " << vecRegionScore[0] << endl;
        cout << "Region Area : " << vecRegionArea[0] << endl;
        cout << "Region Perimeter : " << vecRegionPerimeter[0] << endl;
        cout << "Region Center X : " << vecRegionCenterX[0] << endl;
        cout << "Region Center Y : " << vecRegionCenterY[0] << endl;

        region_node = region_node->next_sibling();
    }
}
{% endhighlight %}

실행결과 아래와 같이 원하는 값을 얻을 수 있습니다.

<img src="{{ site.baseurl }}/assets/posts/2018-07-03-CppRuntimeApi/06.png">

<br/>
#### Overlay 이미지 생성하기
ViDi를 통해 처리된 결과 마킹 - Feature(Blue), Defect 영역(Red) 등 - Overlay 이미지를 생성하려면, `vidi_init_image()` 함수를 통해 초기화하고, `vidi_runtime_get_overlay()` 함수를 통해 Overlay 이미지를 얻고, `vidi_save_image()` 함수를 통해 이미지를 저장하고, `vidi_free_image()` 함수를 통해 이미지 메모리를 반납합니다.

Overlay 이미지는 원본 이미지와 동일한 해상도이므로, 이 이미지를 원본 이미지 위에 오버레이하여 ViDi의 결과 정보를 시각적으로 가시화할 수 있습니다.

<img src="{{ site.baseurl }}/assets/posts/2018-07-03-CppRuntimeApi/07.png">
[원본 이미지]

<img src="{{ site.baseurl }}/assets/posts/2018-07-03-CppRuntimeApi/08.png">
[오버레이된 이미지]
