# ROSbag to PCD

rosbag 파일에서 PCD 파일을 추출 하는 방법은 두가지 입니다.

1. rosbag파일에서 직접 추출 하는 방법과 
2. rosbag파일을 재생 후에 실시간으 추출 하는 방법 입니다. 이 방법은 실시간으로 수집되는 데이터를 PCD로 저장 하기에 유리 합니다. 

## 1. rosbag파일에서 직접 추출 하는 방법

bag파일에서 pcd파일을 추출 하시려면 아래의 명령어를 이용 하면 됩니다.

```text
$ rosrun pcl_ros bag_to_pcd lobby_lidar.bag /velodyne_points ./lobby_pcd
# rosrun pcl_ros bag_to_pcd <input_file.bag> <topic> <output_directory>
```

## 2. rosbag파일을 재생 하여 실시간으로 추출 하는 방법

```python
#!/usr/bin/env python
import sys
import os
import rospy
import numpy as np
import cv2
import pcl
import sensor_msgs.point_cloud2 as pc2
from sensor_msgs.msg import PointCloud2
from cv_bridge import CvBridge

save_path = None

def cloud_loader(msg):
    timestamp = msg.header.stamp.secs + ((msg.header.stamp.nsecs + 0.0) / 1000000000)
    save_pcd(msg, timestamp, save_path)

def save_pcd(cloud, timestamp, path):
    p = pcl.PointCloud(np.array(list(pc2.read_points(cloud)), dtype=np.float32)[:, 0:3])
    p.to_file(path + '/pcd' + '_' + "{:.5f}".format(timestamp) + '.pcd')

def rosbag_data_extract_sample():
    global save_path
    try:
        save_path = sys.argv[1]
        topic = sys.argv[2]
    except Exception, e:
        #sys.exit("Please specify the save path. Example: rosbag_data_extract_unsync.py /media/0/output/")
        save_path = './sample'

    node_name = "get_%s_and_convert_to_PCD_data" % topic
    rospy.init_node('rosbag_pcd_extract_unsync', anonymous=True)

    rospy.Subscriber(topic, PointCloud2, cloud_loader)
    rospy.spin()

if __name__ == '__main__':
    rosbag_data_extract_sample()
```

