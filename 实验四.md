# Taking a Snapshot and Viewing Processes
- - - 
# 课堂实验 查看当前进程
* 实现编程显示进程 ，在[网站](https://docs.microsoft.com/zh-cn/windows/win32/toolhelp/taking-a-snapshot-and-viewing-processes) 中复制代码，增加头文件，修改循环中代码  
![pic1-1](pic/exp1-1.png)
![pic1-2](pic/exp1-2.png)
* 运行代码，实验结果与tasklist命令下显示运行的进程列表相同  
![pic1-3](pic/exp1-3.png)
* 修改代码，当进程名为Theads.exe时，打印详细信息  
![pic1-4](pic/exp2-1.png)
* 观察详细信息结果可以看到，第一个模块一定是exe文件，带了若干个数量不等的dll文件，基础API都在kernel32.dll当中  
![pic1-5](pic/exp2-2.png)
* 在空工程中创建a.c和b.c文件，分别编译后，将a.obj链接b.obj生成haha.exe  
![pic1-6](pic/a.c.png)
![pic1-7](pic/b.c.png)
![pic1-8](pic/obj生成.png)
* 由于b中调用messagebox，链接时加入messagebox函数所需要的库文件user32.lib，程序最终才能运行成功  
![pic1-9](pic/user32.png)
![pic1-10](pic/链接成功.png)

- - - 
# 作业
## 一、实验要求
* 编写dll。把.c文件编译为obj文件，把obj文件和lib文件链接为新的dll和lib文件。注意使用def文件定义导出函数。
* 编写一个exe，调用第一步生成的dll文件中的导出函数。方法是：
    * (1)link时，将第一步生成的lib文件作为输入文件
    * (2)保证dll文件和exe文件在同一个目录，或者dll文件在系统目录
* 上一步调用方式称为load time，特点是exe文件导入表中会出先需要调用的dll文件名及函数名，并且在link生成exe时，需明确输入lib文件。还有一种调用方式称为 run time。参考[链接](https://docs.microsoft.com/zh-cn/windows/win32/dlls/using-run-time-dynamic-linking)使用run time的方式，调用dll的导出函数。包括系统API和第一步自行生成的dll，都要能成功调用

## 二、实验内容
### 1.编写DLL文件
#### (1)用命令行编译链接产生DLL文件  
* 编写base.c和exp.def文件  
![pic2-1](pic/base.c.png)  
![pic2-2](pic/exp模块定义文件.png)
* 编译生成base.obj文件，添加exp模块定义文件链接为DLL文件
![pic2-3](pic/链接生成DLL文件.png)
    * 注意可能产生报错，此时将/out参数删去  
    ```link base.obj /dll /def:exp.def```  
    生成baselib.dll文件

#### (2)用VS直接生成DLL文件  
* 在工程属性——>配置属性——>常规中更改配置类型为```动态库```文件   
![pic2-4](pic/改变配置类型.png) 
* 在工程属性——>配置属性——>链接器——>输入中添加模块定义文件exp.def  
![pic2-5](pic/添加模块定义文件.png) 
* 调试代码，在工程文件夹\baselib\baselib下生成base.lib和daselib.dll文件  
![pic2-6](pic/VS生成DLL文件.png)  

### 2.编写exe文件，调用第一步生成的dll文件中的导出函数
* 编写app.c文件，其中调用baselib.dll文件中的函数lib_funtion()
![pic2-7](pic/app.c.png) 
* 编译app.c文件生成obj文件，链接生成exe文件时添加lib_funtion函数所需要的库文件baselib.lib
![pic2-8](pic/编译链接exe文件.png) 
* 目录下产生app.exe文件，且成功调用了导出函数lib_funtion()
![pic2-9](pic/生成exe文件.png) 
    * 注意：链接过程中需要保证dll文件和exe文件在同一个目录，或者dll文件在系统目录  

### 3.使用run time的方式，调用dll的导出函数
* 在[链接](https://docs.microsoft.com/zh-cn/windows/win32/dlls/using-run-time-dynamic-linking)中复制代码，进行修改,表示使用LoadLibrary函数获得baselib DLL的句柄。如果LoadLibrary成功，程序将使用lib_funtion函数中返回的句柄来获取DLL的lib_funtion函数的地址。调用DLL函数后，程序调用FreeLibrary函数卸载DLL
![pic2-10](pic/rt.c.png) 
* 当dll文件和exe文件在同一个目录时，运行成功
![pic2-11](pic/rt.png) 
* 当dll文件和exe文件不在同一个目录时，报错
![pic2-12](pic/rt报错.png) 


## 三、实验总结
* 编写一个exe，调用dll文件中的导出函数时应该注意：
    * link时，将第一步生成的lib文件作为输入文件
    * 保证dll文件和exe文件在同一个目录，或者dll文件在系统目录

* 运行时加载，可以使程序自己在运行时控制加载的指定的模块，并在不需要使用的时候卸载。其特性如下： 
    * 不必从程序一开始就将其全部装载进来，减少了程序启动时间和内存使用
    * 程序不必重启就可以实现模块的增加、删除、更新等

* 运行时和加载时动态链接之间的一个重要区别：如果DLL不可用，则使用加载时动态链接的应用程序必须终止。但是，运行时动态链接示例可以响应错误。