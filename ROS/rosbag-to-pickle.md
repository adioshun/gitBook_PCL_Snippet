```python 
import sensor_msgs.point_cloud2 as pc2
import numpy as np
import rosbag
import os 
import pickle
import argparse


parser = argparse.ArgumentParser()
parser.add_argument("--bagfile", help="Use baseline rosbag to remove background")
parser.add_argument("--topic", default="/velodyne_points", help="Use baseline rosbag to remove background")
parser.add_argument("--dir", default='./', help="Use baseline rosbag to remove background")
parser.add_argument("--points", default=50, help="Use baseline rosbag to remove background")

args = parser.parse_args()

bag_file = args.bagfile
bag_topic = args.topic
save_dir = args.dir
points = args.points

print("")
print("Load rosbag : {}".format(bag_file))
bag = rosbag.Bag(bag_file, "r")
messages = bag.read_messages(topics=[bag_topic])
n_lidar = bag.get_message_count(topic_filters=[bag_topic])
print("Load Topic: {}".format(bag_topic))

baseline = np.zeros((0,5),dtype=np.float32)
print("Number of Points: {}\n".format(points))
for i in range(points):#n_lidar):
    # READ NEXT MESSAGE IN BAG
    topic, msg, t = messages.next()  #print(dir(msg))

    # CONVERT MESSAGE TO A NUMPY ARRAY OF POINT CLOUDS
    # creates a Nx5 array: [x, y, z, reflectance, ring]
    lidar = pc2.read_points(msg)
    lidar = np.array(list(lidar),  dtype=np.float32)
    baseline = np.concatenate((baseline, lidar),axis=0)
    print("{}-{}".format(i, baseline.shape[0]))

baseline = baseline[:,0:3]

save_file = save_dir + bag_file + ".pkl"
output = open(save_file, 'wb')
pickle.dump(baseline, output)
output.close()
print("File saved : {}\n".format(save_file))
```