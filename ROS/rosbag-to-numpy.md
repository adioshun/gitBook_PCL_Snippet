# ROS Pointcloud2 데이터를 numpy(np)파일로 저장 

> https://github.com/hengck23/didi-udacity-2017/blob/master/baseline-04/didi_data/ros_scripts/run_dump_lidar.py
> 개인적으로 pickle 파일 저장 추천


* [ROS bags-TO-Point Clods.ipynb](https://gist.github.com/anonymous/e675ea14113252be321320be62248034)

---


# rosbag을 Lidar 정보를 npy로 저장하기 

- [Part 1](http://ronny.rest/blog/post_2017_03_29_ros/), [Part 2(이미지처리)](http://ronny.rest/blog/post_2017_03_30_ros2/), [Part 3](http://ronny.rest/blog/post_2017_03_30_ros3_and_lidar/)

참고
- [ROS wiki-rosbag-cookbook](http://wiki.ros.org/rosbag/Cookbook)
- [programcreek-rosbag](https://www.programcreek.com/python/example/89952/rosbag.Bag)


```python
import sensor_msgs.point_cloud2 as pc2
import numpy as np
import rosbag
import os
import yaml
```


```python
# ROSBAG SETTINGS
data_dir = "/workspace/_rosbag/"
bag_name = 'cafeteria_baseline_2018-11-16-03-24-16.bag'
bag_file = os.path.join(data_dir, bag_name)

bag = rosbag.Bag(bag_file, "r")
```

## 1. bag 정보 확인


```python
info_dict = yaml.load(rosbag.Bag(bag_file, 'r')._get_yaml_info())
# Print the information contained in the info_dict
info_dict.keys()
for topic in info_dict["topics"]:
    print("-"*50)
    for k,v in topic.items():
        print(k.ljust(20), v)
```

    --------------------------------------------------
    ('topic               ', '/camera/image_color')
    ('type                ', 'sensor_msgs/Image')
    ('frequency           ', 6.8499)
    ('messages            ', 61)
    --------------------------------------------------
    ('topic               ', '/velodyne_points')
    ('type                ', 'sensor_msgs/PointCloud2')
    ('frequency           ', 9.8882)
    ('messages            ', 126)



```python
!rosbag info /workspace/_rosbag/cafeteria_baseline_2018-11-16-03-24-16.bag
```

    path:        /workspace/_rosbag/cafeteria_baseline_2018-11-16-03-24-16.bag
    version:     2.0
    duration:    12.8s
    start:       Nov 16 2018 11:24:16.48 (1542367456.48)
    end:         Nov 16 2018 11:24:29.25 (1542367469.25)
    size:        652.6 MB
    messages:    187
    compression: none [187/187 chunks]
    types:       sensor_msgs/Image       [060021388200f6f0f447d0fcd9c64743]
                 sensor_msgs/PointCloud2 [1158d486dd51d683ce2f1be655c3c181]
    topics:      /camera/image_color    61 msgs    : sensor_msgs/Image      
                 /velodyne_points      126 msgs    : sensor_msgs/PointCloud2


## 2. Lidar 데이터 처리


```python
messages = bag.read_messages(topics=['/velodyne_points'])
n_lidar = bag.get_message_count(topic_filters=["/velodyne_points"])
```


```python
baseline = np.zeros((0,5),dtype=np.float32)
```


```python
baseline.shape
```




    (0, 5)




```python
lidar.shape
```




    (26942, 5)




```python

for i in range(50):#n_lidar):
    
    # READ NEXT MESSAGE IN BAG
    topic, msg, t = messages.next()  #print(dir(msg))

    # CONVERT MESSAGE TO A NUMPY ARRAY OF POINT CLOUDS
    # creates a Nx5 array: [x, y, z, reflectance, ring]
    lidar = pc2.read_points(msg)
    lidar = np.array(list(lidar), dtype=np.float32)
    #baseline = np.append(baseline, lidar)
    baseline = np.concatenate((baseline, lidar),axis=0)
    print(baseline.size)

```

    6999385
    7133920
    7268630
    7403395
    7538115
    7672735
    7807745
    7942300
    8076895
    8211405
    8346090
    8480760
    8615525
    8750020
    8884715
    9019340
    9153980
    9288640
    9423380
    9557455
    9691765
    9826470
    9961165
    10095730
    10230460
    10365225
    10499920
    10635625
    10770330
    10904955
    11040650
    11175130
    11309660
    11443990
    11576925
    11711305
    11845550
    11979210
    12112085
    12246380
    12380615
    12515175
    12649820
    12784600
    12919325
    13053970
    13188710
    13323305
    13458010
    13593005
    13727655
    13862315
    13996965
    14131645
    14266500
    14401970
    14536650
    14671525
    14806285
    14940930
    15075560
    15210300
    15345070
    15479905
    15614655
    15749750
    15884620
    16019055
    16153425
    16288995
    16423700
    16558350
    16693065
    16827895
    16962690



    ---------------------------------------------------------------------------

    StopIteration                             Traceback (most recent call last)

    <ipython-input-165-db93605ce195> in <module>()
          3 
          4     # READ NEXT MESSAGE IN BAG
    ----> 5     topic, msg, t = messages.next()  #print(dir(msg))
          6 
          7     # CONVERT MESSAGE TO A NUMPY ARRAY OF POINT CLOUDS


    StopIteration: 



```python
baseline.shape
```




    (1346025, 5)



### 2.1 npy 저장

- 3D 데이터 저장후 Load시 문제 발생 --> pickle추천


```python
baseline = baseline[:,0:3]
save_file = bag_name + ".npy"
np.save(save_file, baseline)
```


```python
baseline.shape
```




    (1346025, 3)




```python
lidar
```




    array([[ 6.0014157 , -3.15215   , -1.8163921 ,  6.        ,  0.        ],
           [ 8.684567  , -4.5633674 ,  0.17124301, 72.        ,  8.        ],
           [ 6.9221344 , -3.6388266 , -1.8054571 , 10.        ,  1.        ],
           ...,
           [ 5.5711823 , -3.9650896 ,  1.5787065 ,  5.        , 14.        ],
           [ 8.272041  , -5.8895044 , -0.17724663, 57.        ,  7.        ],
           [ 4.338231  , -3.0898628 ,  1.4271283 , 27.        , 15.        ]],
          dtype=float32)



### 2.2 pickle 저장


```python
import pickle
save_file = bag_name + ".pkl"
output = open(save_file, 'wb')
pickle.dump(baseline, output)
output.close()
```


```python
pkl_file = open(save_file, 'rb')

data1 = pickle.load(pkl_file)
```


```python
type(data1)
```




    numpy.ndarray




```python
!ls -lh
```

    total 1.3M
    -rw-r--r-- 1 root     root      37K Nov 20 04:01 bag2np.ipynb
    -rw-rw-r-- 1 adioshun adioshun  810 Nov 20 02:22 bag2np.py
    -rw-r--r-- 1 root     root     317K Nov 20 03:54 cafeteria_baseline_2018-11-16-03-24-16.bag.npy
    -rw-r--r-- 1 root     root     908K Nov 20 04:00 cafeteria_baseline_2018-11-16-03-24-16.bag.pkl
    -rw-rw-r-- 1 adioshun adioshun 6.3K Nov 20 03:25 main.py
    -rw-rw-r-- 1 adioshun adioshun 8.0K Nov  5 14:57 main_centroid_20191105.py
    -rw-rw-r-- 1 adioshun adioshun 7.0K Nov  5 10:35 main_sort_20191105.py
    -rw-r--r-- 1 root     root      14K Nov  9 07:26 people_tracker.ipynb


### 2.1 시각화 


```python
import sys
sys.path.append("/workspace/include")
from visualization_helper import *

%matplotlib inline
```


```python
visualization2d_arr_xyz(np.load(save_file))
```

    (x) : 92.7m
    (y) : 53.3m
    (z) : 7.4m



![png](output_24_1.png)


## 3. Image 데이터 처리 

- `pc2.read_points(msg)` 대신 `np.fromstring(msg.data, dtype=np.uint8)` 사용 
- 자세한 내용은 [[여기]](http://ronny.rest/blog/post_2017_03_30_ros2/) 참고 










