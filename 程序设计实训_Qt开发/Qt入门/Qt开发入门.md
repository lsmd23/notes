## Qt开发概述

- Qt是一个C++的库，主要用于可视化应用程序设计中，支持cmake构建项目，同时有自己的qmake，是一种面向对象的设计方式，借助**slot和signal机制**实现控件之间的通信，有自己的IDE——QtCreator
## Qt的编译

- 写好的Qt代码可以利用命令行编译
- 在所有程序中搜索Qt的命令行窗口，打开后进行编译：
  1. 使用**qmake -project**命令生成.pro工程文件（需配置环境变量）；
  2. 在文件中加入**QT += widgets gui**命令；
  3. 使用**qmake**命令构建文件；
  4. 使用**mingw32-make**命令构建，即可得到可执行文件；
## QtCreator

- 可以直接构建项目：
    - Details选项下有：**QMainWindow**——带有菜单栏的窗口；**QWidget**——正常窗                              口；**QDialog**——对话框；
    - 可以选择翻译，编译器等等；
- 文件内容：工程文件、头文件、cpp源文件、ui文件（.ui为后缀）
    - 头文件中，声明了Widget作为一个类，具有一个特殊的参数**ui指针**，在构造函数中用this指针初始化，用于访问这个widget下的各种成员
    - main.cpp中有这个窗口运行的必要程序框架
    - 界面文件widget.ui中，可以直接用可视化方法设计ui界面
        - 布局、标签、弹簧、容器等等……
        - 右侧编辑对象名
        - 右下角可以调节参数

- *QWidget中，有调节控件尺寸参数的选项**geometry**，以及调节文本内容字体的选项**font**，右击改变多信息文本可以改变信息*
## 信号与槽

- **信号**：指对控件做出操作后的反应
- **槽**：指接受信号后程序的响应方式（执行的代码）
- 实现方式：
    1. 右击控件，选择**转到槽**，选择信号后即自动合成槽与信号的代码
    2. 在界面的构造函数中，利用关联函数connect()连接信号和槽
        - 四个参数：信号源，信号类型，接收器，槽函数
        - 例：`connect(ui->cancelButton,&QPushButton::clicked,this,&Widget::slot)`
    3. 利用lambda表达式

- *访问界面文件对象：使用ui指针访问控件*
- *对LineEdit文本编辑器，text()函数以QString参数格式返回其内容*
- *this->close()函数关闭窗口*
## 实现四则运算的计算器

- 步骤：
    1. 建立所有的按钮，利用限制最大尺寸和最小尺寸的方式限制按钮尺寸，编辑对象名
    2. 设计ui界面
    3. 分别实现槽函数

- ***网格布局**：网格状布局，选中后布局即可*
- *LineEdit控件中，setText()函数接受一个QString类型的参数，用于显示在文本框中*
- *setWindowTitle()接受一个字符串，用于设置窗口标题，在构造函数中使用*
- *在按钮上显示图片：借助QIcon对象实现，其接受目标图片所在的文件路径的字符串作为参数，实现对象的构造，在借助button控件的setIcon()函数，接受该对象为参数构造*
- *也可以直接在按钮选项中在theme下构造*
- *QString类型中有一个chop(int n)成员函数可以去掉末尾n个字符*
- *styleSheet选项可以改变按钮的颜色、样式等等*
## QObject定时器

- QObject类本身就提供了定时器功能的成员函数
- 开启定时器：`this->startTimer()`方式启动定时器，传入一个参数计时，以毫秒为单位
- 定时器事件：计时到后程序的响应函数，在QObject中是一个虚函数，需要在头文件中重写虚函数来实现：`virual void timerEvent(QTimerEvent *event)`，它传入的参数为当前定时器的编号
    - 定时器编号：一个代码里面可以有数个定时器，不同定时器依靠编号不同来区分，其中：`this->startTimer()`函数的返回值就是定时器的编号，为int类型
    - 一般在程序头文件中用一个变量储存
```
//Widget.h
	int myTimerID;
//Widget.cpp
	myTimerID = this->startTimer();
```
- 实现函数时，为防止不同定时器混用，需先判断定时器编号是否与记录下的编号一致
```
if(event->timerId() != myTimerID)
    return;
/*TO DO*/ 
```
- 关闭定时器：`this->killTimer()`函数，传递定时器编号即可

- *在标签上显示图片：*
    - *使用QPixmap对象，用路径初始化，再使用`label->setPixmap(QPixmap pix)`来显示*
    - *注：scaledContents选项可以让图片自动适配标签的大小*
## QTimer定时器

- QTimer是Qt提供的定时器类，满足定时器的功能
- 使用时，需要先`include <QTimer>`包含所在头文件，然后在头文件定义其指针`QTimer *timer`;再在构造函数中用new方法初始化；
- 开启定时器直接用其`startTimer(TIME)`函数开启
- QTimer的机制是计时，到时间后发出一个timeout信号，响应方式用槽函数的方式定义，因此需在构造函数中使用connect函数链接信号与槽
```
//Widget.h
private slots:
	void timeoutSlot();
//Widget.cpp
	connect(timer, &QTimer::timeout, this, &Widget::timeoutSlot);
```
- 关闭定时器：成员函数`stop()`即可关闭定时器
- 其他功能：有使得定时器仅工作一次的成员函数`singleShot(1, 2, 3)`，其有三个参数，第一个参数是等待时间，第二个参数是接收信号的对象，第三个参数是处理的槽函数
```
QTimer::singleShot(1000, this, SLOT(timeoutSlot()));
```
- 另有其他的功能，相对更便捷

- *在标签上显示图片：*
    - *使用QImage对象，用load方法初始化，再用fromImage方法加载为Pixmap对象*
```
QImage img;
img.load("路径")；
ui->label->setPixmap(QPixmap::fromImage(img));
```
## Qt文件操作

