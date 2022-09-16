# Android-group-chat-room-based-on-socket

**一、目标**

1. 掌握基于 Android 的网络编程的相关知识，实现群聊

1. 学会如何定义网络协议。

1. 学会如何传输表情、图片等特殊信息。

1. 手机照相、相册机权限请求

**二、结构图**

1、服务器与客户端

serverSocket和socket的连接
<img width="337" alt="image" src="https://user-images.githubusercontent.com/71966043/190598786-d655849e-35d9-4e71-a100-752c4cc13077.png">

2、ServerThread和ClientThread介绍
<img width="309" alt="image" src="https://user-images.githubusercontent.com/71966043/190598835-332034e8-cb26-4b8d-876a-859508f5834e.png">

**三、程序机制介绍**

**关于界面**

1、ListView

这个控件也是让我走了一个很大的坑，首先就是List中的每一项（item）的布局可能是不同的

1.1只发送文字消息时

模拟器接收消息真机发送消息
<img width="205" alt="image" src="https://user-images.githubusercontent.com/71966043/190598871-a2088e3d-46c6-40f8-af3e-33f3a86fe07a.png"><img width="197" alt="image" src="https://user-images.githubusercontent.com/71966043/190598880-2730c697-a0a7-4af3-8fbb-2aaec5d460fb.png">


1.2只发送图片消息时

真机发送图片模拟器接收图片

<img width="194" alt="image" src="https://user-images.githubusercontent.com/71966043/190598935-b2e233e8-fd40-46c7-ab8e-3fb9c87d50c8.png"><img width="204" alt="image" src="https://user-images.githubusercontent.com/71966043/190598941-99acaf55-88fc-4293-ad0d-2b8c2ab5bc31.png">


1.3发送文本消息加图片消息

真机接受文本和图片消息模拟器发送文本和图片消息

<img width="200" alt="image" src="https://user-images.githubusercontent.com/71966043/190598966-1859d8e8-70a2-4c84-8228-128158a9cdd5.png"><img width="214" alt="image" src="https://user-images.githubusercontent.com/71966043/190598977-68b34eb4-e9f7-48d4-92a6-c5299ba56802.png">


所以说ListView中的每一行（每一个item）一共可能会出现六种布局
<img width="213" alt="image" src="https://user-images.githubusercontent.com/71966043/190599051-cdfd8785-7868-411e-b391-7f4c6d00d336.png">

根据getItemViewType()来获取Information中的消息类型
<img width="408" alt="image" src="https://user-images.githubusercontent.com/71966043/190599128-63462597-6e18-4029-930e-1a2049590636.png">
<img width="395" alt="image" src="https://user-images.githubusercontent.com/71966043/190599139-5cc83567-282b-45d9-a81a-4e5bcbd2abef.png">

getView()中根据type = getItemViewType()来设置item布局

<img width="408" alt="image" src="https://user-images.githubusercontent.com/71966043/190599148-699ca0f5-224b-45e5-9f4a-659c93dbc3a5.png">

（小部分getView()代码）
<img width="415" alt="image" src="https://user-images.githubusercontent.com/71966043/190599163-2674e0d6-c347-4a83-a2e0-f0d5b7c2f102.png">

2、客户端UI主线与ClientThread子线程的交互

因为网络连接和读取消息是有堵塞的可能的，然而我们并不能让UI主线程堵塞，因为用户需要滑动和点击来查看和发送消息，如果UI主线程堵塞，在用户的角度看就像程序卡死了一样，用户体验会很低，所以在Android中，我们用Handle机制来解决这一问题

UI主线程中

<img width="256" alt="image" src="https://user-images.githubusercontent.com/71966043/190599181-17eb6f0a-ecf4-478e-a640-83c3ce18869b.png">

网络子线程ClientThread中：

派生一个子线程用来读取服务器发送过来的消息对象，并发送消息对象到消息队列中

<img width="316" alt="image" src="https://user-images.githubusercontent.com/71966043/190599204-8a27949a-7021-4c5e-b348-8b8789d7cff3.png">

UI主线程中的handle对消息队列中的消息进行处理，更新ListView
<img width="264" alt="image" src="https://user-images.githubusercontent.com/71966043/190599221-9ba2ea1d-aaaf-4f7a-8b61-cc6423896874.png">

UI主线程中:

当要发送文字消息将用户姓名、时间、文字消息封装成Information对象，将消息发送到receiveHandler的消息队列中

<img width="305" alt="image" src="https://user-images.githubusercontent.com/71966043/190599236-0bb28df7-23f9-4434-988e-fd6baaed7cc4.png">

网络子线程将UI主线程发送过来的Informaiton对象用writeObject()发送到服务器上，因为writeObject()

不会堵塞，所以这里要用loop机制
<img width="393" alt="image" src="https://user-images.githubusercontent.com/71966043/190599260-e4afb203-2822-4781-a4a4-9f447a69edfd.png">
**关于动态申请权限问题**

权限在AndroidManifest.xml文件中声明，Android 6.0以前，有的APP一股脑声明了各种各样的权限，用户可能没有细看就安装了，于是这些APP就可以为所欲为。Android 6.0把权限分成正常权限和危险权限，AndroidManifest中声明的正常权限系统会自动授予，而危险权限则需要在使用的时候用户明确授予。

本程序中的读取相册和调用手机相机就属于危险权限

<img width="134" alt="image" src="https://user-images.githubusercontent.com/71966043/190599364-d4f20a37-a73e-487b-b7bc-47236374627f.png">

1.需要申请的所有权限先在AndroidManifest文件中声明
<img width="429" alt="image" src="https://user-images.githubusercontent.com/71966043/190599443-ce399168-d2ab-4c0a-8d4f-2f60504605eb.png">

危险权限需要在代码中动态申请

需要赋予程序危险权限时，checkSelfPermission()检查该权限是否已经被授予

如果没被授予，调用requestPermissions（）

<img width="342" alt="image" src="https://user-images.githubusercontent.com/71966043/190599467-6e9d5838-062c-4018-9f99-5391859fa058.png">

弹出一个权限请求对话框

用户点击允许或者禁止后，程序会调用onRequestPermissionsResult()来检查用户是否授予程序申请的危险权限
<img width="451" alt="image" src="https://user-images.githubusercontent.com/71966043/190599486-e1a24d06-88d8-4317-8da8-0ff2710944eb.png">

grantResults数组就是申请的危险权限，数组长度就是一次性申请的危险权限的数量

此时如果用户允许ChatRoom访问相册，程序就会调用openAlbum()
<img width="451" alt="image" src="https://user-images.githubusercontent.com/71966043/190599509-cb6d2c49-399b-401f-9eb3-6cfb1a6d1254.png">

以上就是Android基于socket网络聊天程序的比较重要的几点

效果截图

<img width="139" alt="image" src="https://user-images.githubusercontent.com/71966043/190599530-e68c240e-1eec-46d3-b951-868f708e9ce4.png"><img width="140" alt="image" src="https://user-images.githubusercontent.com/71966043/190599581-b2d736bb-9a25-4d64-aef8-c590ad584917.png"><img width="164" alt="image" src="https://user-images.githubusercontent.com/71966043/190599605-433d400f-446d-4ee0-b780-003570e5a176.png">

<img width="207" alt="image" src="https://user-images.githubusercontent.com/71966043/190599621-d4889e74-a0de-4e61-9699-a005108f1393.png"><img width="206" alt="image" src="https://user-images.githubusercontent.com/71966043/190599629-3009bf73-e52b-41fc-8563-c185d360bf2f.png">



心得体会

因为时间问题，程序中依然有一些bug未解决，比如发送客户端和服务器不在同一个内网（局域网）时发送图片会非常的慢。

还有一些自己想要添加的功能没有实现，如私聊，用户注册，并将注册信息保存至服务器数据库，传输文件，音频文件，视频文件等，以及emoji表情的发送并展示等等。
