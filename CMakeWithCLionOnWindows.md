# CMake与第三方库导入-以Libuv与CLion工程为例

对于VS工程，使用CMake构建之后按照：https://learn.microsoft.com/zh-cn/cpp/build/walkthrough-creating-and-using-a-dynamic-link-library-cpp?view=msvc-170 这个链接操作即可
对于使用CMake的CLion工程，还需要进行导入与修改CMakeLists的几步操作

# Build With VS
在安装了Visual Studio之后，CMake GUI的默认编译器为VS，导出的动态运行时库通常会生成两个文件：一个是.lib（库导入文件），另一个是.dll（运行库本身）。这两个文件在动态链接过程中起着不同的作用

## .lib  库导入文件
- .lib文件是库导入文件，用于在编译时与应用程序进行链接。它包含了对动态运行时库中函数和符号的引用
- .lib文件包含了函数和数据的引用信息，但实际的代码和数据存储在.dll文件中
- 在编译时，链接器使用.lib文件来解析对库中函数和符号的引用，并在最终的可执行文件中建立正确的链接关系

## .dll  运行库本身
- .dll文件是运行库本身，包含了实际的可执行代码和数据
- .dll文件在应用程序运行时被动态加载和链接
- 当应用程序启动时，它会加载所需的.dll文件，并在运行时通过函数调用来访问其中的代码和数据
- 动态运行时库的使用可以实现代码的共享，减少可执行文件的大小，并允许多个应用程序共享同一份库

## include文件夹  运行库的头文件集
- include文件夹中包含了所需的头文件，严格来说，这不算是VS导出的文件，但仍然需要放入项目目录中以使用库中的符号

.lib文件用于在编译时与库进行链接，而.dll文件包含了实际的可执行代码和数据，在运行时动态加载和链接。这种分离的设计允许应用程序与动态运行时库保持独立，并在需要时加载所需的库文件。

# 导入CLion中

## 1.导入文件
在根目录下新建bin、lib与include文件夹，将uv.dll放入bin文件夹，将uv.lib放入lib文件夹，将需要的头文件放入include文件夹（一般由第三方库提供）
## 2.修改CMakeLists.txt
``` MakeFile
#请自行修改ProjectName，lib与dll文件的路径、名字等信息

include_directories(include) #包含头文件目录

target_link_libraries(ProjectName ${CMAKE_CURRENT_SOURCE_DIR}/lib/uv.lib) #对库导入文件的引用

add_custom_command(TARGET LibuvTest POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
        ${CMAKE_CURRENT_SOURCE_DIR}/bin/uv.dll
        $<TARGET_FILE_DIR:ProjectName>) #生成时将DLL文件复制到目录中，供程序进行动态链接

```

至此，我们便能在CLion中使用第三方库
