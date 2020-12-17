# Index정보 사용하기

// Color-based region growing segmentation // [http://pointclouds.org/documentation/tutorials/region\_growing\_rgb\_segmentation.php\#region-growing-rgb-segmentation](http://pointclouds.org/documentation/tutorials/region_growing_rgb_segmentation.php#region-growing-rgb-segmentation)

결과 값으로 Index정보를 얻어 사용

```cpp
  pcl::IndicesPtr indices (new std::vector <int>);  // 중요
  pcl::PassThrough<pcl::PointXYZRGB> pass;
  pass.setInputCloud (cloud);
  pass.setFilterFieldName ("z");
  pass.setFilterLimits (0.0, 5.0);  //mistypo changed from 1 to 5
  pass.filter (*indices);  // 중요

  pcl::RegionGrowingRGB<pcl::PointXYZRGB> reg;
  reg.setInputCloud (cloud);
  reg.setIndices (indices);   // 중요 
  reg.setSearchMethod (tree);
  reg.setDistanceThreshold (10);
  reg.setPointColorThreshold (6);
  reg.setRegionColorThreshold (5);
  reg.setMinClusterSize (600);

}

---
포인트 정보 

  pcl::search::Search <pcl::PointXYZRGB>::Ptr tree (new pcl::search::KdTree<pcl::PointXYZRGB>);
  pcl::PointCloud <pcl::PointXYZRGB>::Ptr cloud (new pcl::PointCloud <pcl::PointXYZRGB>);
  pcl::io::loadPCDFile <pcl::PointXYZRGB> ("region_growing_rgb_passthrough.pcd", *cloud);

  pcl::RegionGrowingRGB<pcl::PointXYZRGB> reg;
  reg.setInputCloud (cloud);
  //reg.setIndices (indices);   
  reg.setSearchMethod (tree);
  reg.setDistanceThreshold (10);
  reg.setPointColorThreshold (6);
  reg.setRegionColorThreshold (5);
  reg.setMinClusterSize (600);
```

