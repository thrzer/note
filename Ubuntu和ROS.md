by：thrzer

- [Ubuntu](#ubuntu)
  - [路径](#路径)
  - [文件操作](#文件操作)
  - [编程文件操作](#编程文件操作)
  - [其他](#其他)
    - [帮助](#帮助)
- [ROS](#ros)
  - [节点与节点管理器](#节点与节点管理器)
    - [节点Node](#节点node)
    - [节点创建](#节点创建)
    - [节点管理器ROS master （仅在ros1，ros2没有这个）](#节点管理器ros-master-仅在ros1ros2没有这个)
  - [其他](#其他-1)
    - [多线程](#多线程)
    - [循环](#循环)
  - [功能包](#功能包)
    - [一般安装](#一般安装)
    - [手动编译安装](#手动编译安装)
    - [功能包相关指令](#功能包相关指令)
      - [xml文件](#xml文件)
      - [cfg文件](#cfg文件)
  - [工作空间](#工作空间)
    - [colcon使用](#colcon使用)
  - [通信（未完待续）](#通信未完待续)
    - [话题topic 发布/订阅](#话题topic-发布订阅)
    - [补充：msg、srv和action文件](#补充msgsrv和action文件)
      - [msg文件](#msg文件)
      - [srv文件](#srv文件)
    - [服务service 请求/应答](#服务service-请求应答)
  - [其他工具](#其他工具)
    - [ros2 bag](#ros2-bag)
    - [RQT工具](#rqt工具)
    - [RVIZ2](#rviz2)
    - [Gazebo](#gazebo)
- [Micro-ROS](#micro-ros)
- [FreeRTOS](#freertos)
# Ubuntu
<font size = 4>**xxx表示位置，name表示文件名**</font>

很多软件和包安装不了可能是源出了问题，可以尝试小鱼的一键换源（学长提供）
```
wget http://fishros.com/install -O fishros && . fishros
```

## 路径
查看当前路径
```
pwd
```
前往路径
```
cd /xxx
```
返回上一位置
```
cd ..
```
## 文件操作
文件操作都是基于本路径下的文件

创建文件夹
```
mkdir name
当前目录下创建
mkdir -p name1/name2
递归创建
```
删除文件夹
```
rm -r name
目录及其内容都会被删除
rm -d name
删除空文件夹
```
创建文件
```
touch name
```
删除文件
```
rm name
```
查看该路径下的内容
```
ls
```
剪切文件
```
mv name /xxx
```
复制文件
```
cp name /xxx/newname
如果不写“/newname”意为不新起名
```
## 编程文件操作
编译C++文件
```
g++ name.cpp -o name
用g++编译器把name.cpp文件编译成name.o文件
```
运行C++文件
```
./name
```
解析并运行python文件
```
python3 name.py
```
## 其他

### 帮助
一般命令行中，都可以使用--help或-h查看帮助
```
xx --help
xx -h
```
ros中新的功能包使用前常需要设置工作环境，可以用source命令执行对应文件，如：
```
source /xxx/setup.bash
```

# ROS
<font size = 4>全名 Robot Operating System</font>\
本篇为关于ros2的笔记，主要基于“鱼香ROS2”的讲解

<font size = 3 color = #00ffff>首先可以查阅</font>
<a href = "https://docs.ros.org/en/humble/index.html">ROS官方文档</a>\
<font size = 3 color = #00ffff>或者C++的</font>
<a href = "https://docs.ros.org/en/humble/p/rclcpp/generated/index.html">API文档</a>（注意版本，这里是humble）

## 节点与节点管理器
所有相关命令都可以以输入ros2 xxx(某一命令)来查看可选内容，获得帮助
### 节点Node
ROS中一个功能模块，通常是实现单个功能

节点需要在管理器进行注册\
多个节点共同形成了节点计算图\
节点通过DDS协议互相联系

运行节点：
```
ros2 run <功能包名称> <节点名称>
```
查看运行中节点：
```
ros2 node list
```
查看节点内容：
```
ros2 node info </节点名称>
```
节点重映射：
```
ros2 run <功能包名称> <节点名称> _ros_args --remap __node:=<重映射的新名称>
```

### 节点创建
<font size = 4 color = #afff00>第一步（其一）：</font>\
创建功能包之后，在src中新建cpp文件，在cpp文件中编写节点代码

```
/*
编写R0S2节点的一般步骤
1.导入库文件
2.初始化客户端库
3.新建节点对象
4.spin循环节点
5.关闭客户端库
*/
#include "rclcpp/rclcpp.hpp"    //导入库文件
using namespace std;

int main(int argc,char ** argv)
{
    rclcpp::init(argc,argv); //初始化客户这库
    auto node1 = make_shared<rclcpp::Node>("nodename"); //新建节点对象，并起名（千万别往名字里加下划线 “_” ）

    RCLCPP_INFO(node1->get_logger(),"随便写点啥当作日志");  //打印内容

    rclcpp::spin(node1);    //spin循环节点
    rclcpp::shutdown(); //关闭客户端库
}
```

<font size = 4 color = #afff00>第一步（其二）：</font>
如果需要构建节点类的话，可以参照这样的示例：
```
class NodeName:public rclcpp::Node  //继承节点类型rclcpp::Node
{
public:
    NodeName(std::string name):Node(name)
    {
        RCLCPP_INFO(this->get_logger(),"这里是%s，随便写点啥当作日志",name.c_str());
        //name.c_str()返回的是接收到的数据
    }
}
```

这样的话，main函数中新建节点就要做对应的修改：
```
    auto node1 = make_shared<NodeName>("nodename");
    //auto会自动推导应有的类型
```
这里的nodename是外部查看时的节点名称，比如通过node list看到的就是这个

<font size = 4 color = #afff00>第二步：</font>\
进入cmakelist.txt文件进行配置，在末尾后添加以下内容，这样就可以主动编译文件

```
# 第一个节点
add_executable(node_name src/node1.cpp)
# 为节点对应可执行文件，比如node_name节点对应的执行文件是node1.cpp
ament_target_dependencies(node_name rclcpp)
# 添加依赖，所有添加的库文件都要加上！用空格隔开
install(TARGETS
  node_name
  DESTINATION lib/${PROJECT_NAME}
)
# 多个节点创建需要重复以上内容
```
注意这里面的node_name是终端用来run的节点名称，建议和cpp文件中起的那个相同

<font size = 4 color = #afff00>第三步：</font>\
最后到终端编译该文件
```
写完记得保存所有文件！！！
colcon build --packages-select testnode
用之前记得source一下
source install/setup.bash
可以在节点目录查看
ros2 pkg list | grep node
用“node”过滤查找
ros2 run testnode node_name
最后可以运行试试
```

### 节点管理器ROS master （仅在ros1，ros2没有这个）
将各个节点之间联系起来，通常是管理数据的收发\
节点管理器可以存一些静态变量，例如一些参数，供节点使用（全局字典）

## 其他
### 多线程
main默认是单线程函数，有时候单线程会产生死锁的情况，需要用多线程处理\
在main函数中添加以下内容创建多线程：
```
rclcpp::excutors::MultiThreadedExecutor executor;
executor.add_node(node1);
executor.spin();
```

### 循环
可按照如下方法进行一秒一次的循环：
```
rclcpp::Rate rate(1);
while(/*...*/)
{
  /*...*/
  rate.sleep();
}
```

## 功能包
### 一般安装
```
sudo apt install ros-<ros版本>-<功能包名称>
```
### 手动编译安装
用colcon（略）

### 功能包相关指令
功能包创建：
```
ros2 pkg create <功能包名称> --build-type <编译类型，cmake/ament_cmake/ament_python>  --dependencies <依赖功能包名称>
编译类型可以省略，默认ament_cmake
依赖部分可以也不写，在后续添加
```
完成这一步后，会自动创建一些文件，可以在src文件中加入新节点

功能包列出：
```
ros2 pkg list
列出所有包
ros2 pkg executables
列出所有可执行文件
ros2 pkg executables <功能包名称>
列出功能包下的可执行文件
```
功能包路径：
```
ros2 pkg prefixe <功能包名称>
查看功能包路径
```

#### xml文件
xml文件是功能包的描述清单，记录了包的名字、依赖包、编译类型、作者、版本、构建工具、功能等信息，便于使用者查阅编译顺序、安装依赖等\
查看清单文件：
```
ros2 pkg xml <功能包名称>
```

#### cfg文件
cfg文件与python功能包的工作空间有关，是一种配置文件，用于配置节点的参数，例如节点名称、话题名称、服务名称等 `这一点不是很重要`

## 工作空间
一般的工作空间，都包含build，install，log，src四个文件夹

### colcon使用
用colcon编译工作空间
```
colcon bulid
需要处于对应工作空间的目录下，即该功能包对应src目录的上一级
```
只编译一个功能包
```
colcon build --packages-select <功能包名称>
```
对应python功能包，只有colcon build其实只是拷贝了一份过去，如果希望不需要每次更改都拷贝，可以使用链接的手段
```
colcon build --symlink-install
```
<font size = 3 color = #00ffff>更多内容可见</font>
<a href = "https://colcon.readthedocs.io/en/released/user/quick-start.html">colcon官方文档</a>


## 通信（未完待续）
主要有话题和服务两种通信方式

### 话题topic 发布/订阅
异步，由一个节点发布数据，其他节点订阅数据

查看正在发布的话题：
```
ros2 topic list
ros2 topic list -t 查看更多信息
```
进行订阅（这是一种对信息的截获）：
```
ros2 topic echo </话题名称>
```
发布订阅：
```
ros2 topic pub <进行订阅的对象></话题名称><消息类型><数据>
订阅对象可以不写（可能订阅者之后才出现）
举例：
ros2 topic pub /topic_name std_msgs/String 'data: "Hello World"'
```
查看话题信息：
```
ros2 topic info </话题名称>
会显示接口的参数类型等
ros2 topic interface </话题名称>
显示消息的具体内容
```

<font size = 4 color = #afff00>创建订阅者：(subscriber.hpp)</font>

```
/*
编写R0S2话题订阅者的一般步骤
1.导入消息类型（库文件）
2.创建回调函数
3.声明和创建订阅者
4.编写回调处理逻辑
*/

#include "rclcpp/rclcpp.hpp"
#include "std_msgs/msg/string.hpp"
//导入库文件、消息类型

using std::placeholders::_1;
//引入占位符

class nodetype : public rclcpp::Node
{
private:
    //声明订阅者
    rclcpp::Subscription<std_msgs::msg::String>::SharedPtr suber_name;
    //因为订阅者可能被其他节点调用，所以需要使用共享指针

    //创建回调函数
    void text_callback(const std_msgs::msg::String::SharedPtr msg_from_pub)
    {
        //编写回调处理逻辑
        RCLCPP_INFO(this->get_logger(),"here's%s,随便写点啥当作回复",msg_from_pub->data.c_str());
    }
    
public:
    nodetype(std::string name):Node(name)
    {
        RCLCPP_INFO(this->get_logger(),"here's%s,随便写点啥当作日志",name.c_str());

        //创建订阅者
        suber_name = this->create_subscription<std_msgs::msg::String>("topic_name",10,std::bind(&nodetype::text_callback,this,_1));
        /*参数表为：话题名称，队列长度，回调函数。
        因为成员函数不能直接用于回调，所以需要用bind，这是C++11的新特性，能用于复制一份新函数。
        参数表为：函数地址，通过哪个对象调用（传入地址）,占位符（取决于函数有几个参数）。*/
    }
};
```
注意这里的topic_name要和发布者一致

<font size = 4 color = #afff00>创建发布者：(publisher.hpp)</font>

```
/*
编写R0S2话题发布者的一般步骤
1.导入发布者的消息接口类型
2.声明和创建发布者
3.发布消息
*/

#include "rclcpp/rclcpp.hpp"
#include "std_msgs/msg/string.hpp"
#include "std_msgs/msg/u_int32.hpp"
//导入发布者的消息接口类型

class nodetype : public rclcpp::Node
{
private:
    rclcpp::Publisher<std_msgs::msg::String>::SharedPtr puber_name;
    //声明发布者
public:
    nodetype429b(std::string name):Node(name)
    {
        //创建发布者
        puber_name = this->create_publisher<std_msgs::msg::String>("topic_name",10);
        //发布消息
        std_msgs::msg::UInt32 i;
        std_msgs::msg::String msg;
        for(i.data = 0;;i.data++)
        {
            RCLCPP_INFO(this->get_logger(),"here's puber_name,我发布了%d",i.data);
            msg.data = "不知道说啥好";
            puber_name->publish(msg);
            sleep(1);
        }
    }
};
```
注意发布者的消息类型要和订阅者一致\
<font size = 3 color = #ff7f00>同一个节点可以既是发布者，又是订阅者，把两部分的内容都加上即可</font>

### 补充：msg、srv和action文件

#### <font size = 4>msg文件</font>
话题传输的数据内容叫做消息message，消息通常有特定的数据结构\
消息的传输可以自定义的接口，用msg文件定义\
可以通过命令行查看msg接口：
```
ros2 interface list

ros2 interface package <包名称>
ros2 interface packages

ros2 interface show <完整接口名称>
查看接口定义
ros2 interface proto <完整接口名称>
查看具体属性
```

用户可以自定义msg文件：\
<font size = 4 color = #afff00>第一步：</font>\
创建一个msg文件，这里起名为Testtopic.msg

```
# 调用ros2中的原始数据类型
string content
# 调用功能包sensor_msgs中已有的数据类型
sensor_msgs/Image image

# 说明这个自定义接口包含字符串和图像两种数据类型
```
msg文件名称第一个字母必须<font size = 3 color = #ff7f00>**大写**</font>！！

<font size = 4 color = #afff00>第二步：</font>\
定义好的msg文件要由Cmake文件转化成C++的头文件，进入代码中，因此还要修改cmakelist.txt文件：

```
# 添加必要的依赖
find_package(sensor_msgs REQUIRED)
find_package(rosidl_default_generators REQUIRED)

rosidl_generate_interfaces(${PROJECT_NAME}
  "msg/Testtopic.msg"
  DEPENDENCIES sensor_msgs
)
# 以上内容不必写在结尾，也可以写在其他find_package后面
```
PS:这部分不写不会报错，但运行的时候就会出问题了

<font size = 4 color = #afff00>第三步：（可选）</font>\
修改package.xml文件：

```
<buildtools_depend>ament_cmake</buildtools_depend>
……
<!-- 写在上面这句话之后(xml语言这样注释) -->
<depend>sensor_msgs</depend>
<build_depend>rosidl_default_generators</build_depend>
<exec_depend>rosidl_default_runtime</exec_depend>
<member_of_group>rosidl_interface_packages</member_of_group>
……
```
命令行里source之后使用其他命令查看msg文件，若能正常显示说明接口定义成功

#### <font size = 4>srv文件</font>
自定义的接口，用srv文件定义

<font size = 4 color = #afff00>第一步：</font>\
建立srv文件：（同样名称首字母大写，这里起名为Testservice.srv）
```
# 请求
string content
uint32 num
---    # 请求与响应类型间用三个 “-” 隔开
# 响应
bool issuccess
uint32 num
```

<font size = 4 color = #afff00>第二步：</font>\
修改cmakelist.txt文件：
```
rosidl_generate_interfaces(${PROJECT_NAME}
  "msg/Testtopic.msg"
  # 在前面的基础上再加上srv的依赖
  "srv/Testservice.srv"
  DEPENDENCIES sensor_msgs
  # 因为这里srv用的都是原始数据类型，所以不需要加其他内容
)
```

<font size = 4 color = #afff00>第三步：</font>\
修改package.xml文件：

```
<!-- 同样是因为srv没有其他依赖，这里不做其他修改 -->
<!-- 如果有其他依赖，可以参考msg的修改方式做出修改 -->
```

### 服务service 请求/应答
同步，一个节点发送请求，另一个节点应答，并发送数据

查看服务列表：
```
ros2 service list
```
查看服务接口：
```
ros2 service type </服务名称>
查看类型
ros2 interface show </服务名称>
查看接口内容
ros2 service find </接口名称>
查找使用该接口的服务（运行中的）
```
请求服务：
```
ros2 service call <请求服务的对象></服务名称> <服务类型> <数据>
```


## 其他工具
### ros2 bag
还有一些以ros2 bag开头的命令
操作录制：可以将用户的操作录制下来，进行复现
```
ros2 bag record
```

播放时，可以应用倍数播放：
```
ros2 bag play <文件名> -r <倍数>
```

还可以单曲循环：
```
ros2 bag play <文件名> -r <倍数>
```

### RQT工具
RQT工具是ROS2的图形化工具，可以查看节点关系图，查看节点状态，查看节点日志，查看节点参数等

在终端中打开RQT
```
rqt
```

可以使用RQT工具查看节点关系图
```
ros2 run node.py listener
ros2 run node.cpp talker
假如已经运行了两个节点

rqt_graph
查看关系图
```
（进去之后要先刷新一下才显示出来）

测试服务时，在上方工具栏选择service，输入内容，点击call即可

除此以外RQT工具还有很多功能，这里不详细列举了

### RVIZ2
一个数据可视化工具

进入RVIZ2：
```
rviz2
```

### Gazebo
一个仿真模拟工具

进入Gazebo：
```
gazebo
```

需要通过ros2的话题服务等控制仿真，这里以驱动小车前进为例
```
gazebo /opt/ros/humble/share/gazebo_plugins/worlds/gazebo_ros_diff_drive_demo.world
召唤小车
ros2 topic pub /demo/cmd_demo geometry_msgs/msg/Twist '{linear: {x: 0.5, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 0.0}}'
输入速度，开始前进
```



# Micro-ROS
基于微控制器（ESP32，STM32，Arduino等）的控制平台

MicroRos支持各种通信协议，但需要依赖Agent。微控制器通过串口、蓝牙、WIFI等协议与Agent进行通信，Agent通过网络协议，转化为话题等数据，与ROS2进行通信

MicroRos的代码编写使用C语言，用到的API是rclc
```
（测试用，待完善）
sudo docker run -it --rm -v /dev:/dev -v /dev/shm --privileged --net=host microros/micro-ros-agent:$ROS_DISTRO serial --dev /dev/ttyUSBO -v6
```

# FreeRTOS
real time operating system.\
FreeRTOS是开源的实时操作系统，主要应用在微处理器上，具有高度的实时性，并且支持多任务，多线程，多核等特性。\
<font size = 3 color = #00ffff>查阅</font>
<a href = "https://freertos.org">FreeRTOS官方文档</a>
