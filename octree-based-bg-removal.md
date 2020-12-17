# Octree를 이용한 배경 제거

```python
#!/usr/bin/env python3
# coding: utf-8

import sys
sys.path.append("/workspace/include")

from ddynamic_reconfigure import DDynamicReconfigure 

import rospy
from sensor_msgs.msg import PointCloud2


import numpy as np
import pcl
import pcl_msg

import pcl_helper


import argparse
import pickle


class DyConfigure(object):
    def __init__(self):
        # Create a D(ynamic)DynamicReconfigure
        self.ddr = DDynamicReconfigure("BgRemoval")

        # Add variables (name, description, default value, min, max, edit_method)

        self.ddr.add_variable("resolution", "float/double variable", 0.1, 0.0, 1.0)
        self.add_variables_to_self()
        self.ddr.start(self.dyn_rec_callback)

    def add_variables_to_self(self):
        var_names = self.ddr.get_variable_names()
        for var_name in var_names:
            self.__setattr__(var_name, None)

    def dyn_rec_callback(self, config, level):
        rospy.loginfo("Received reconf call: " + str(config))
        # Update all variables
        var_names = self.ddr.get_variable_names()
        for var_name in var_names:
            self.__dict__[var_name] = config[var_name]
        return config


class BgRemoval:
    def __init__(self):
        self.lidar = rospy.Subscriber(args.topic, PointCloud2, self.callback) 
        self.pub_bg = rospy.Publisher("/velodyne_bg_201", PointCloud2, queue_size=1)   
        self.searchPoint = pcl.PointCloud(pickle.load(open(args.baseline, 'rb'))) #da_play_baseline_201.pkl"
        self.E = DyConfigure()

    def background_removal(self, daytime, nighttime):

        resolution = self.E.resolution #1.0#0.8  # 값이 커지면 missing 존재, noise도 존재 
        #배경 포인트 
        octree = nighttime.make_octreeChangeDetector(resolution)
        octree.add_points_from_input_cloud ()
        octree.switchBuffers () #Switch buffers and reset current octree structure.

        # 입력 포인트 #cloudB cloudA

        daytime = pcl_helper.XYZRGB_to_XYZ(daytime)

        octree.set_input_cloud(daytime)
        octree.add_points_from_input_cloud ()
        newPointIdxVector = octree.get_PointIndicesFromNewVoxels ()
        daytime.extract(newPointIdxVector)


        #배경 제거 포인트 
        result = np.zeros((len(newPointIdxVector)+1, 3), dtype=np.float32)
        for i in range(0, len(newPointIdxVector)):
            result[i][0] = daytime[newPointIdxVector[i]][0]
            result[i][1] = daytime[newPointIdxVector[i]][1]
            result[i][2] = daytime[newPointIdxVector[i]][2]

        pc = pcl.PointCloud(result)           
        cloud = pcl_helper.XYZ_to_XYZRGB(pc,[255,255,255])

        return cloud

    def callback(self, input_ros_msg):       
        pcl_xyzrgb = pcl_helper.ros_to_pcl(input_ros_msg) 
        pcl_xyz = pcl_helper.XYZRGB_to_XYZ(pcl_xyzrgb)

        pcl_xyzrgb = self.background_removal(pcl_xyz, self.searchPoint)

        bg_ros_msg = pcl_helper.pcl_to_ros(pcl_xyzrgb)      
        self.pub_bg.publish(bg_ros_msg)


if __name__ == '__main__':
    rospy.init_node('BackgroundRemoval', anonymous=True)

    parser = argparse.ArgumentParser()
    parser.add_argument("--baseline", default="./baseline_201.pkl", help="Use baseline rosbag to remove background")
    parser.add_argument("--topic", default="/lidar_201/velodyne_points", help="Input Topic")


    args = parser.parse_args()

    print("\nLoad baseline(--baseline) : {}".format(args.baseline))
    print("Input Topic(--topic) : {}".format(args.topic))    
    print("Ouput Topic(FIXED) : /velodyne_bg_201")

    bg = BgRemoval()
    rospy.spin()
```

