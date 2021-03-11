作为开源计算机视觉库，opencv在多个平台上被使用是常有的需求。本文将根据官方文档，简要给出在windows平台与linux平台上使用opencv的c++ API并构建项目的指引。

**适用范围**：需要在Microsoft VS下使用opencv的c++ API，且在**链接时**动态链接与加载opencv动态库。本文不涉及从0开始构建opencv库，使用opencv静态库，或运行时动态使用opencv动态库。这方面的内容可以参考[官方教程](https://docs.opencv.org/master/df/d65/tutorial_table_of_content_introduction.html)。

## 获取Pre-built Libraries

直接访问GIthub上opencv的[releases](https://github.com/opencv/opencv/releases)地址，选择需要的版本下载即可。在windows环境下上请下载`opencv-[version]-cv14_vc15.exe`一项，下载内容是所谓的self-extracting archive。将它放在你期望的目录下，以管理员身份运行，当前目录下会出现`opencv`的文件夹。

在opencv文件夹下，我们只需要关心如下三个事情：

- 在`/build/include`中的各种头文件，只有在项目设置中告诉IDE此位置，IDE才有可能做出opencv API相关的语法检测与拼写提示，也是正确build项目的必要条件。
- 位于`/build/x64/vc15/bin`下的动态链接库`opencv_world[version].dll`与`opencv_world[version]d.dll`，**前者用于release模式下的项目构建，后者用于debug模式**。在一些opencv的版本中，此库分成了多个独立的动态链接库。
- 位于`/build/x64/vc15/lib`下的后缀为.lib的文件`opencv_world[version].lib`。同样，在一些opencv的版本中，此文件分成了多个独立的.lib文件。

需要注意，位于`/build/x64/vc15/lib`下的后缀为.lib的文件与静态编译产生的.lib文件是不同的概念。此处的文件被称作import library，在VS构建项目，此文件将提供给链接器一些用到的dll文件的附加信息。如果是GNU的linker则不需要这些，更详细的内容请看[这里](https://stackoverflow.com/questions/3573475/how-does-the-import-library-work-details)。

## 添加环境变量

在链接时，系统会到path变量中的路径下搜索使用的动态链接库。因此，我们需要将上文提到的`opencv_world[version].dll`所在路径添加到path变量中。

具体的，分为两步。

- 首先新建系统变量`OPENCV_DIR`,取值为build文件夹的路径，如`C:\LIB\opencv\build`。这个变量可以简化后面的很多工作。

- 在Path变量中新建路径，此路径是bin文件的位置，如`%OPENCV_DIR%\x64\vc15\bin`。

## 设置Microsoft VS

接下来就是对IDE的设置了，更具体的说，是对编译选项和链接选项的设置，而这些设置都会反映到VS项目中的属性文件`[PropertySheetName].props`中（VS2019版本的名称）。下面的步骤假设使用VS2019版本，**构建平台为x64**，使用Debug模式。

设置共分为三步。

- 新建项目，点击`视图`$\rightarrow$`属性窗口`，打开属性管理器。右键`Debug|x64`选择`添加新项目属性表`，将属性表合理命名，并添加。展开`Debug|x64`一栏，将看到刚刚添加的属性表文件。如下图所示：

  ![propertySheet](https://raw.githubusercontent.com/OkifuZ/images/master/propertySheet.png)

- 双击打开属性表，展开`C/C++`，在`常规`中设置`附加包含目录`，添加include文件的路径，如`$(OPENCV_DIR)\include;`。此时，可以引用opencv的头文件而不至于报错。

  ```cpp
  #include <opencv2/core.cpp> // no error should occur
  ```

- 还是在属性表中，展开`链接器`。

  - 在`常规`中设置`附加库目录`，添加.lib文件所处路径，如`$(OPENCV_DIR)\x64\vc15\lib;`。
  - 在`输入`中设置`附加依赖项`，指定dll文件对应的.lib文件，如我的opencv**只有一个**Debug模式用的dll文件`opencv_world[version]d.dll`，那么只需要添加对应的.lib文件即可：`opencv_world[version]d.lib`。*如果下载的opencv版本有多个dll，而没有以`opencv_world`开头的dll，则需要分别添加对应的.lib文件。*

  这样就把opencv的动态链接库的import library文件告诉了VS的链接器，VS因此获得了正确寻找与链接opencv库到你写的程序中的功能。

这里我展示出我的属性表配置文件`PropertySheet.props`的具体内容，这个文件可以复制到任何需要使用opencv的c++项目中，因此，保存这个文件，在新建项目时拷贝过去即可，而不再需要重新设置VS。

```xml
<?xml version="1.0" encoding="utf-8"?>
<Project ToolsVersion="4.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <ImportGroup Label="PropertySheets" />
  <PropertyGroup Label="UserMacros" />
  <PropertyGroup />
  <ItemDefinitionGroup>
    <ClCompile>
      <AdditionalIncludeDirectories>$(OPENCV_DIR)\include;%(AdditionalIncludeDirectories)</AdditionalIncludeDirectories>
    </ClCompile>
    <Link>
      <AdditionalLibraryDirectories>$(OPENCV_DIR)\x64\vc15\lib;%(AdditionalLibraryDirectories)</AdditionalLibraryDirectories>
      <AdditionalDependencies>opencv_world451d.lib;%(AdditionalDependencies)</AdditionalDependencies>
    </Link>
  </ItemDefinitionGroup>
  <ItemGroup />
</Project>
```

注意这里的系统变量`OPENCV_DIR`，这样做比使用绝对路径的好处在于，**可以将此文件应用于任何设置了`OPENCV_DIR`变量的机器上，而不用关心opencv的具体路径在哪里。**类似的变量有设置Java环境时使用的`JAVA_HOME`，设置CUDA环境时使用的`CUDA_PATH_V[version]`。

## 测试

如果成功编译运行下面的代码，说明您已经可以在windows上正确使用opencv c++ API了。

```cpp
#include <opencv2/core.hpp>
#include <opencv2/imgcodecs.hpp>
#include <opencv2/highgui.hpp>
#include <iostream>
#include <string>

using namespace cv;
using namespace std;

int main() {
	string PIC_BASE = "C:\\picture_base";
	Mat image;
	image = imread(PIC_BASE + "\\test.png", IMREAD_COLOR);
	if (image.empty()) {
		cout << "Could not open or find the image" << endl;
		return -1;
	}
	namedWindow("Display Window", WINDOW_AUTOSIZE);
	imshow("Display Window", image);
	waitKey(0);
	return 0;
}
```

运行结果将展示路径在`C:\\picture_base\\test.png`下的图片：

![blade runner](https://raw.githubusercontent.com/OkifuZ/images/master/bladeRunner_AI.png)



END