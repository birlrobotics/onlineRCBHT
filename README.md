基于相对变化的在线层次语义分类系统
=======

本系统可以将实时的力螺旋信息(wrench data)在线编译成语义符号。其输入是实时的、包含力螺旋信息的机器人操作系统（ROS）信息，然后用在线并行处理方法将其编码后，将编码好的语义符号发送回ROS网络。

输入：
    接收来自ROS topic名称为/robot/limb/right/endpoint_state的数据。
输出：
    ROS消息类型：（自定义，需自行安装）
    segmentMsg      ： publish_files/Segments
    compositeMsg    ： publish_files/Composites
    llbehaviorMsg   ： publish_files/llBehaviors
    
    计算结果，分六个维度通过ROS Topic发送：
    Fx, Fy, Fz, Mx, My, Mz
    
    发送与Fx相关计算结果的ROS Topic:
    /topic_segments_Fx        消息类型（publish_files/Segments）
    /topic_composites_Fx      消息类型（publish_files/Composites）
    /topic_llbehaviors_Fx     消息类型（publish_files/llBehaviors）
    发送与Fy相关计算结果的ROS Topic:
    /topic_segments_Fy        消息类型（publish_files/Segments）
    /topic_composites_Fy      消息类型（publish_files/Composites）
    /topic_llbehaviors_Fy     消息类型（publish_files/llBehaviors）
    Topic的命名规则是将维度名称替换，以此类推，可知发送与Fz/Mx/My/Mz相关语义符号的ROS Topic及其对应的消息类型。
    
软硬件要求：
     1. 本系统是并行系统，若想真正地发挥出其性能，请务必在核数>=8的计算机上运行。
     2. 本系统基于ubuntu 14.0，Matlab R2015b，使用到的Matlab工具箱有机器人系统工具箱（Robotic System Toolbox）,并行计算工具箱(Parallel Computing Toolbox)，请确保已安装好上述软件。
    
    
环境配置：
     1. 为了在matlab发送自定义的ROS消息，需要在已经安装了机器人系统工具箱的基础上，再附加安装一个ROS自定义消息支持包，安装方法如下：
            1.1 若matlab版本是R2015b, 在matlab命令窗口运行命令： roboticsSupportPackages 并跟随其指令进行安装。
            1.2 若matlab版本是R2016a及以上, 在matlab命令窗口运行命令： roboticsAddons 并跟随其指令进行安装。
     2. 为了在ROS网络上接收计算结果，需安装ROS包“publish_files”，这个包有两个功能，i)定义了可以承载计算结果（RCBHT Label）的ROS消息，ii)可以把格式化的力螺旋信息文件转变成信息流发送到ROS网络。
            2.1 将/publish_folder文件夹拷贝到你的ROS workspace的src文件夹下。
            2.2 回到ROS workspace目录下运行catkin_make.
            2.3 至此，可以承载计算结果的自定义消息已可以使用，验证方法： $ rosmsg show publish_files/Segments。
            2.4 若需要将文件转变成信息流： $ rosrun publish_Files force_publisher [path_of_force_file]
      
3. Generate custom message for RCBHT's use.
      3.1 Specify a folder path where your want to place your custom messages in. Such as: 
                  folderpath = '/home/drinkcor/MATLAB/custom_msg'
      3.2 Put folder: publish_files/ under the folder path that you defined above.
      3.3 Generate custom message: rosgenmsg(folderpath)

4. Be sure that you have added all needed codes to the Path:
      i)   the folder of online RCBHT
      ii)  the folder of custom message support
      iii) the folder of generated custom message

运行online RCBHT:
1. 在终端运行 $ roscore
2. 在matlab命令行开启并行计算池：>> parpool(8); （注：若并行计算池已开启，则不必再次运行此命令）。
3. 在matlab命令行输入命令：>> [Results] = SharedM_onlineSnapVerification;


To use online RCBHT, there are several steps listed below,

1. Start ROS, 
      $ roscore

2. Open parpool in MATLAB command window.
      % Delete any possible existing pools running previously
      >> delete(gcp);
      >> parpool(2);
  Then, wait for the parpool start. If the prompt, which is ">>"， available again, that means the parpool 
  is ready to work. Now we can move on to the next step.

3. Run RCBHT in MATLAB command window.
      >> rt_snapVerification('HSA','any_path_here_is_ok');
   "HSA" means a kind of strategy, the second parameter is a path, some files containing predefined theshold 
   are placed here. If there aren't any predefined threshold files, you can text any string to fill this 
   parameter. After you see message like: “The value of the ROS_MASTER_URI environment variable, 
   http://localhost:11311, will be used to connect to the ROS master.Initializing global node 
   /matlab_global_node_88464 with NodeURI http://localhost:60491/” It mean that the RCBHT is ready to work. 
   Move on to the next step.
   
4. Publish real time force/torque data to RCBHT.
   The RCBHT receive real time force/torque data. To publish real time data to it, we have several approach: 
      1) connect to a robot using ROS system, 
      2)run a rosbag, 
      3)publish a force/torque file to be real time data. 
   
   4.1 If use approach 2), just run,
            $ rosbag play -r 0.4 [path/xxx.bag]
       Here, -r 0.4 means multiply rate with 0.4. (If you want to know the rate of a topic, you can use 
            $ rostopic hz /topic_name.)
       Then, the online RCBHT will receive data from rosbag and process it.
   4.2 If use approach 3), you need to have publishFiles package in your ros workspace, so that you can run 
       force_publisher node.After you installing publishFiles, run,
            $ rosrun publishFiles force_publisher [path_of_force_file]
       Then, the online RCBHT will receive data from a real time data(originnally is a file) and process it.  
     
online RCBHT (real time RCBHT)
=======================

Overview:

The online RCBHT enables semantic encoding of low-level real time wrench data. It takes real 
time ROS message (includeing wrench data) as input, do semantic encoding, and finally publishs stream 
of semantic labels using ROS message. The taxonomy is built on the premise that low-level relative-change 
patterns can be classified through a small set of categoric labels in an increasingly abstract manner. 
The RCBHT is a multi-layer behavior aggregating scheme. It is composed of three bottom-to-top increasingly 
abstract layers. Starting from the bottom layer and going up we have the Primitive layer, the Motion 
Composition (MC) layer, and the Low-Level Behavior layer (LLB).


Prerequisite：
0. This is a parallel system, in order to fully demonstrate its capability, you should run this system on a computer with 8 or   more cores.
1. Install matlab ROS custom message support. 
      1.1 If your matlab's version is R2015b, call roboticsSupportPackages and follow the instructions 
          for installation. 
      1.2 If your matlab's version is R2016a and later versions, call roboticsAddons and follow the 
          instructions for installation.
      
2. Install ROS package "publish_files". This package can transform file data to be real time data, and 
   publish it to RCBHT. Besides, this package contains the required msg, srv, and package.xml files for 
   RCBHT's use. 
      2.1. Put folder: publish_files/ to the src folder of your ros workspace
      2.2. Run catkin_make outside src folder
      2.3. To use it, run the command: $ rosrun publish_Files force_publisher [path_of_force_file]
      
3. Generate custom message for RCBHT's use.
      3.1 Specify a folder path where your want to place your custom messages in. Such as: 
                  folderpath = '/home/drinkcor/MATLAB/custom_msg'
      3.2 Put folder: publish_files/ under the folder path that you defined above.
      3.3 Generate custom message: rosgenmsg(folderpath)

4. Be sure that you have added all needed codes to the Path:
      i)   the folder of online RCBHT
      ii)  the folder of custom message support
      iii) the folder of generated custom message


To use online RCBHT, there are several steps listed below,

1. Start ROS, 
      $ roscore

2. Open parpool in MATLAB command window.
      % Delete any possible existing pools running previously
      >> delete(gcp);
      >> parpool(2);
  Then, wait for the parpool start. If the prompt, which is ">>"， available again, that means the parpool 
  is ready to work. Now we can move on to the next step.

3. Run RCBHT in MATLAB command window.
      >> rt_snapVerification('HSA','any_path_here_is_ok');
   "HSA" means a kind of strategy, the second parameter is a path, some files containing predefined theshold 
   are placed here. If there aren't any predefined threshold files, you can text any string to fill this 
   parameter. After you see message like: “The value of the ROS_MASTER_URI environment variable, 
   http://localhost:11311, will be used to connect to the ROS master.Initializing global node 
   /matlab_global_node_88464 with NodeURI http://localhost:60491/” It mean that the RCBHT is ready to work. 
   Move on to the next step.
   
4. Publish real time force/torque data to RCBHT.
   The RCBHT receive real time force/torque data. To publish real time data to it, we have several approach: 
      1) connect to a robot using ROS system, 
      2)run a rosbag, 
      3)publish a force/torque file to be real time data. 
   
   4.1 If use approach 2), just run,
            $ rosbag play -r 0.4 [path/xxx.bag]
       Here, -r 0.4 means multiply rate with 0.4. (If you want to know the rate of a topic, you can use 
            $ rostopic hz /topic_name.)
       Then, the online RCBHT will receive data from rosbag and process it.
   4.2 If use approach 3), you need to have publishFiles package in your ros workspace, so that you can run 
       force_publisher node.After you installing publishFiles, run,
            $ rosrun publishFiles force_publisher [path_of_force_file]
       Then, the online RCBHT will receive data from a real time data(originnally is a file) and process it.
```

