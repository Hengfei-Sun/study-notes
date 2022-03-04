# 代码结构

Fast-planner下有三个文件夹，其中主要是fast_planner文件夹，包含规划模块的代码；uav_simulator则是搭建仿真环境；files里存Markdown文件的一些动图，与代码实现无关。

fast_planner文件夹包含建图、前端路径搜索、后端轨迹优化的代码.

[Fast-Planner的github](https://github.com/HKUST-Aerial-Robotics/Fast-Planner)

## 1.代码入口

参考Quick Start 

`source devel/setup.bash && roslaunch plan_manage rviz.launch`

`source devel/setup.bash && roslaunch plan_manage kino_replan.launch`

第一个rviz.launch是打开rviz，展示可视化界面。

第二个kino_replan.launch是打开规划节点和参数的launch文件。位于plan_manage功能包的launch文件夹下。我们入口在这个文件，按顺序阅读即可，

kino_replan.launch下
`include file="$(find plan_manage)/launch/kino_algorithm.xml`

kino_algorithm.xml下
`node pkg="plan_manage" name="fast_planner_node" type="fast_planner_node" output="screen`

节点源码为plan_manage/src/fast_planner_node,开始阅读源码！（因为源码跳转比较多，务必用VS CODE或者其他代码阅读器阅读)



```   
	nh.param("planner_node/planner", planner, -1);
	
	TopoReplanFSM topo_replan;
	KinoReplanFSM kino_replan;

	if (planner == 1) {
    	kino_replan.init(nh);
  	} else if (planner == 2) {
    	topo_replan.init(nh);
  	}
```

planner的值从ros参数服务器获取，在kino_algorithm.xml文件中提供，为1。

TopoReplanFSM topo_replan、KinoReplanFSM kino_replan构造两个类对象，类构造函数实现为空，不用管它。

KinoReplanFSM类的kino_replan对象执行init函数初始化。跳转到函数void KinoReplanFSM::init(ros::NodeHandle& nh)实现。

```
	current_wp_  = 0;
	exec_state_  = FSM_EXEC_STATE::INIT;
	have_target_ = false;
	have_odom_   = false;

	/*  fsm param  */
	nh.param("fsm/flight_type", target_type_, -1);
	nh.param("fsm/thresh_replan", replan_thresh_, -1.0);
	nh.param("fsm/thresh_no_replan", no_replan_thresh_, -1.0);

	nh.param("fsm/waypoint_num", waypoint_num_, -1);
	for (int i = 0; i < waypoint_num_; i++) {
		nh.param("fsm/waypoint" + to_string(i) + "_x", waypoints_[i][0], -1.0);
		nh.param("fsm/waypoint" + to_string(i) + "_y", waypoints_[i][1], -1.0);
		nh.param("fsm/waypoint" + to_string(i) + "_z", waypoints_[i][2], -1.0);
	}
```

由于参数繁多，暂不做解释，等到在函数里使用时再具体介绍，之后的参数设置也先忽略。



```
	planner_manager_.reset(new FastPlannerManager);
  	planner_manager_->initPlanModules(nh);
  	visualization_.reset(new PlanningVisualization(nh));
```
planner_manager_是在KinoReplanFSM类里定义的FastPlannerManager::Ptr对象，为unique_ptr，指向FastPlannerManager对象，通过reset方法指向新生成的FastPlannerManager对象，此指向的对象负责规划的所有流程。

visualization_同上，负责发布可视化msg，在rviz中显示。

planner_manager_->initPlanModules(nh)跳转到函数void FastPlannerManager::initPlanModules(ros::NodeHandle& nh) 实现。

```
  sdf_map_.reset(new SDFMap);
  sdf_map_->initMap(nh);
  edt_environment_.reset(new EDTEnvironment);
  edt_environment_->setMap(sdf_map_);
```
终于快要进入正题了！介绍过planner_manager_后，那应该也能知道sdf_map_是一个智能指针，指向SDFMap对象。那么为什么这边SDFMap使用的是shared_ptr,而之前FastPlannerManager使用的是unique_ptr呢？暂不影响流程推进，继续往下。

sdf_map_->initMap(nh)跳转到void SDFMap::initMap(ros::NodeHandle& nh)实现，我们进入正题，开始初始化地图！


## 2.建图

`node_ = nh`

设置ros节点，回退可以发现是fast_planner_node.cpp里最早定义的ros::NodeHandle nh("~")，ros参数获取，订阅和发布消息依赖于节点。

```
	mp_.local_bound_inflate_ = max(mp_.resolution_, mp_.local_bound_inflate_);
	mp_.resolution_inv_ = 1 / mp_.resolution_;
	mp_.map_origin_ = Eigen::Vector3d(-x_size / 2.0, -y_size / 2.0, mp_.ground_height_);
	mp_.map_size_ = Eigen::Vector3d(x_size, y_size, z_size);
```

mp_.local_bound_inflate_暂时不用理解，用到再说。
	
mp_.resolution_是栅格地图分辨率，即每个栅格的边长，mp_.resolution_inv_是其倒数。

mp_.map_origin_是地图边缘，x最小值，y最小值，和地板高度，单位为m。

mp_.map_size_为地图大小。

SDFMap类的成员变量MappingParameters mp_;MappingData md_;分别为地图参数和地图数据。
	
所有参数都可以在kino_replan.launch文件中或者kino_algorithm.xml文件中根据需求设置。

```
	mp_.prob_hit_log_ = logit(mp_.p_hit_);
	mp_.prob_miss_log_ = logit(mp_.p_miss_);
	mp_.clamp_min_log_ = logit(mp_.p_min_);
	mp_.clamp_max_log_ = logit(mp_.p_max_);
	mp_.min_occupancy_log_ = logit(mp_.p_occ_);
	mp_.unknown_flag_ = 0.01;
```

这里用到了logit()函数，表征栅格占据的概率，概念可参考[Logit究竟是个啥？——离散选择模型之三](https://zhuanlan.zhihu.com/p/27188729),现在不懂没关系，等下用到再去查资料。

`for (int i = 0; i < 3; ++i) mp_.map_voxel_num_(i) = ceil(mp_.map_size_(i) / mp_.resolution_)`
计算体素网格数目，ceil()函数向上取整。

```
	mp_.map_min_boundary_ = mp_.map_origin_;
	mp_.map_max_boundary_ = mp_.map_origin_ + mp_.map_size_;

	mp_.map_min_idx_ = Eigen::Vector3i::Zero();
	mp_.map_max_idx_ = mp_.map_voxel_num_ - Eigen::Vector3i::Ones();
```
设置地图边界，之后有两种概念的边界需要理解，一种是以m为单位的实际地图边界，一种是索引边界，0到体素各轴索引最大值。

`int buffer_size = mp_.map_voxel_num_(0) * mp_.map_voxel_num_(1) * mp_.map_voxel_num_(2)`

设置缓冲大小，设置buffer_size为体素（栅格）总数。

```
  md_.occupancy_buffer_ = vector<double>(buffer_size, mp_.clamp_min_log_ - mp_.unknown_flag_);
  md_.occupancy_buffer_neg = vector<char>(buffer_size, 0);
  md_.occupancy_buffer_inflate_ = vector<char>(buffer_size, 0);
```
md_.occupancy_buffer_为表示栅格占据的vector，初始化为buffer_size个logit（0.12）-0.01；

下面其他容器初始化容器大小都为buffer_size，即栅格总个数，不做过多介绍。

```
	depth_sub_.reset(new message_filters::Subscriber<sensor_msgs::Image>(node_, "/sdf_map/depth", 50));

  	if (mp_.pose_type_ == POSE_STAMPED) {
   		pose_sub_.reset(
        	new message_filters::Subscriber<geometry_msgs::PoseStamped>(node_, "/sdf_map/pose", 25));

    	sync_image_pose_.reset(new message_filters::Synchronizer<SyncPolicyImagePose>(
        SyncPolicyImagePose(100), *depth_sub_, *pose_sub_));
   		sync_image_pose_->registerCallback(boost::bind(&SDFMap::depthPoseCallback, this, _1, _2))
	  }
```

订阅相机深度图和相机位姿信息，注意这里的pose是相机的位姿，在后面depthPoseCallback转换为md_.camera_pos_。

------------------------------写笔记也太难了-----待续