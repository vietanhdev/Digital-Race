<center>
<img src="./media/cover.png" alt="Cover"/>
</center>

### Demo:

<center>
<img src="./media/demo.gif" width="242" height="137"/>
</center>

Welcome! This is an open-source self-driving car aimed for rapid prototyping, deep learning, and robotics research. The system currently runs on a jetson tx2 module. Here are our goals:

### Goals:
Research and develop a deep learning-driven self-driving car. The vehicle should be able to finish the race. 

<center>
<img src="./media/map.jpg" alt="Map" width="300" height="375"/>
</center>

#### The role: 
<center>
<img src="./media/role.jpg" alt="Role" width="300" height="300"/>
</center>

1. Biên, đường đi của xe có thể được xác định ngay cả trong trường hợp có nhiễu và địa hình đường đi
phức tạp (đường có vạch kẻ đường hoặc kẻ nét đứt).

2. Xe khoanh vùng, xác định và tránh được vật cản xuất hiện trên đường (đặc biệt là vật cản động)

3. Xử lý được các tín hiệu nhiễu: Bóng cây, Tuyết trên đường

4. Tất cả các xe thi đấu đều gắn sensor cảm biến và camera ghi lại hành trình thi đấu, chiếu trực tiếp cho
khán giả theo dõi.

5. Trường hợp xe lao ra ngoài: 2 bánh vẫn được chấp nhận, bánh thứ 3 đè lên vạch line 2 bên đường xe phải quay về điểm xuất phát và đi lại.

6. Trên đường 2 chiều, xe luôn phải đi bên làn phải. Xe được phép chuyển làn khi đủ 2 điều kiện sau: vượt xe đi cùng chiều và vạch chia làn đường là nét đứt. Nếu

7. không thực hiện đúng luật xe phải quay về điểm xuất phát và đi lại.

8. Trường hợp xe có thể xin retry: đi sai đường, xe lao ra khỏi đường, xe dừng lại quá lâu.

9. Tại checkpoint 5, xe phải dừng đủ 3 giây và 2 bánh xe trước đè lên hoặc vượt qua vạch xuất phát. Trường hợp xe không dừng đủ 3 giây, mốc hoàn thành xa nhất
của xe là mốc cảm biến thứ 4, trước mốc STOP.

#### The modules in this project.

1. Semantic Segmentation
2. Object Detection
3. Mapping with rtabmap
4. Path planning with ROS nav stack. 

For the full documentation of the development process, please visit my website: [datvuthanh.github.io](https://datvuthanh.github.io)

## Table of Content
- [Introduction](#Introduction)
- [Try it out](#Try%20it%20out)
- [About ROS](#About%20ROS)
- [Simulation](#Simulation)
- [Autopilot & End-to-End Behavioral Cloning](#autopilot--end-to-end-behavioral-cloning)
	* [What's Behavioral Cloning](#whats-behavioral-cloning)
	* [Model](#Model)
- [Semantic Segmentation](#Semantic%20Segmentation)
- [The Navigation Stack](#the-navigation-stack)
	* [RTABMap](#RTABMap)
	* [Path Planning](#Path%20Planning)
	* [Vehicle Motion Control](#Vehicle%20Motion%20Control)

<a name="Introduction"></a>
## Introduction
to do it
<a name="Try%20it%20out"></a>

## Try it out
Before you jump in, let me describe the hardware requirement for this project. **A webcam is the minimum requirment.** At this point, you can only run the whole system on the actual self-driving vehicle. ROS will throw warnings (even errors) at you if you don't have the hardware connected to your Linux machine. **If you don't have access to the hardware setup, don't worry** 👇
 
- The best way is to download and play back the ROS bags. [coming soon...]
- You can tryout individual packages and nodes, and might find them helpful for your own projects. 
- You can also tryout the CARLA simulator. (Maybe even improve the current system.)

To compile the project:

##### Requirements

1. Make sure that you have [ROS](http://wiki.ros.org/melodic/Installation/Ubuntu) installed on your computer. (I am using ROS Melodic)
2. Make sure you have all the [dependencies](./ros/requirements.txt) installed. 

##### Clone & Compile

1. Clone the repository. `$ git clone https://github.com/sigmaai/self-driving-golf-cart.git`
2. `$ cd self-driving-golf-cart/ros` 
3. `$ catkin_make`
4. `$ source devel/setup.bash`

##### Launch ZED cameras
- `$ roslaunch zed_wrapper zed.launch` (no rviz)
- `$ roslaunch zed_display display.launch` (with rviz)

##### Launch the Navigation Stack
- `$ roslaunch path_planning rtab_mapping_navigation.launch` 

<center>
	<img src="./media/path_planning_2.png" alt="image" width="640"/>
</center>

🚙 Bon Voyage 😀

<a name="About%20ROS" > </a>

## About ROS
This project uses ROS. __For more information on ROS, nodes, topics and others please refer to the ROS [README](./ros/README.md).__

<a name="Simulation" > </a>

## Simulation
(🏗 Construction Zone 🚧)

Building a self-driving car is hard. Not everyone has access to expensive hardware. I am currently trying to integrate this project with the CARLA self-driving simulator. If you are interested in CARLA, please refer to this [documentation](./ros/src/simulation/README.md). The ROS system in this project can *partially* run on the CARLA simulator. 

__If you want to try out the simulator, please refer to the documentation [here](./ros/src/simulation/README.md).__

<center>
    <img src="./ros/src/simulation/assets/simulator-1.png" alt="Drawing" width="640"/>
</center>

<a name="autopilot--end-to-end-behavioral-cloning" ></a>

## Autopilot & End-to-End Behavioral Cloning

The autopilot system, found here in the [autopilot node](./ros/src/autopilot), uses deep learning to predict the steering commands and acceleration commands for the vehicle, only using data collected by the front facing camera. 

<a name="whats-behavioral-cloning" > </a>

### What's Behavioral Cloning
In 2016, NVIDIA proposed a novel deep learning approach allowed their car to accurately perform real-time end-to-end steering command prediction. Around the same time, Udacity held a challenge that asked researchers to create the best end-to-end steering prediction model. Our goal is to further the work in behavioral cloning for self-driving vehicles. 

<a name="Model" > </a>

### Model

NVIDIA's paper used a convolutional neural network with a single frame input. I believe that the single-frame-input CNN doesn't provide any temporal information which is critical in self-driving. This is the motive behind choosing the i3d architecture, which is rich in spacial-temporal information.

The input of the network is a 3d convolutional block, with the shape of `n * weight * height * 3`. `n` is the length of the input sequence. A flatten layer and a dense layer are added to the back of the network for the purpose of this regression problem. 

<center>
	<img src="./media/model.png" alt="Drawing" width="640"/>
</center>

Here is a video demo of deep learning model running on the autonomous golf cart. 

[VIDEO DEMO](https://www.youtube.com/watch?v=4bZ40W4BGoE)

<a name="Semantic%20Segmentation" > </a>

## Semantic Segmentation
The cart understands its surrounding  through semantic segmentation, which is a technique in computer that classifies each pixel in an image into different categories. The vehicle can also make decisions based on the segmentic segmentation results. The cart can change its speed based on the proximity to nearby obstacles.

<center>
<img src="./media/seg.png" alt="Drawing" width="640"/>
</center>

We deployed the ENet architecture for segmentation. ENet is design to work well in realtime applications. For more information, please visit the [paper](http://arxiv.org/pdf/1606.02147.pdf). We used the CityScape dataset for training and the python code for training and inferencing are located in the `./src/segmentation/scripts` directory.

[VIDEO DEMO](https://www.youtube.com/watch?v=_y2RCakRrc4)

<a name="the-navigation-stack" > </a>

## The Navigation Stack

![](./media/nav_stack.png)

The self-driving vehicle uses a modified version of the ROS navigation stack. The flowchart above illustrate the mapping and path planning process. First, I create a detailed map of the environment with `rtabmap_ros`. With that global map, I use the localization feature of `rtabmap_ros` and the odom feature of the zed camera system to localize and plan paths. 

<a name="RTABMap" > </a>

### RTABMap

`rtabmap` (realtime appearance based mapping) allows me to construct a global map of the environment. For more information on the mapping package, please check out this [`.launch` file](./ros/src/navigation/mapping/launch/rtab_mapping.launch). 

<center>
	<img src="./media/rtab-map.png" alt="Drawing" width="640"/>
</center> 

<a name="Path%20Planning" > </a>

### Path Planning

The project uses the [`move_base`](http://wiki.ros.org/move_base) node from the navigation stack. The image below shows the costmap (in blue and purple), and the global occupancy grid (in black and gray). `move_base` also plans the local and global path. Global paths are shown in green and yellow below. You can find the `yaml` files [here](./ros/src/navigation/path_planning/params). 

<center>
	<img src="./media/path_plan_1.png" alt="Drawing" width="640"/>
</center>

<a name="Vehicle%20Motion%20Control" > </a>

### Vehicle Motion Control

The move base node publishes `/cmd_vel` commands, which are processed and sent directly to the vehicle. 

<center>
	<img src="./media/vehicle_side.png" alt="Drawing" width="640"/>
</center>

# Contact / Info
If you are interested in the detailed development process of this project, you can visit Neil's blog at [neilnie.com](https://neilnie.com) to find out more about it. Neil will make sure to keep you posted about all of the latest development on the club.

**Contributors:**

**Dat Vu (Leader)** | [Email](mailto:stephen.t.vu@hotmail.com) | [Github](https://www.github.com/datvuthanh) | [Website](https://datvuthanh.github.io/)

<img src="./media/datvu.jpg" alt="Drawing" width="80"/>

**Huy Phan** | [Email](mailto:HuyPQHE141762@fpt.edu.vn) 

<img src="./media/huyphan.png" alt="Drawing" width="80"/>

**Tra Dinh** | [Email](mailto:trandhe140661@fpt.edu.vn) 

<img src="./media/tradinh.png" alt="Drawing" width="80"/>

**Hai Anh Tran** | [Email](mailto:anhthhe141545@fpt.edu.vn) | [Github](https://github.com/AnhTH-FUHN)

<img src="./media/haianh.jpg" alt="Drawing" width="80"/>

