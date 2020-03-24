색상 변경 

```cpp
	pcl::io::loadPCDFile<pcl::PointXYZRGB>("cloud_cluster_0.pcd", *cloud);

     // 시각적 확인을 위해 색상 통일 (255,255,255)
	for (size_t i = 0; i < cloud->points.size(); ++i){
		cloud->points[i].r = 255;
		cloud->points[i].g = 255;
		cloud->points[i].b = 255;
	}
  ```
  
  ```cpp
  	std::vector<int> pointIdxVec;  //결과물 포인트의 Index 저장(Save the result vector of the voxel neighbor search) 

	if (octree.voxelSearch(searchPoint, pointIdxVec))
	{
		//시각적 확인을 위하여 색상 변경 (255,0,0)
		for (size_t i = 0; i < pointIdxVec.size(); ++i){
			cloud->points[pointIdxVec[i]].r = 255;
			cloud->points[pointIdxVec[i]].g = 0;
			cloud->points[pointIdxVec[i]].b = 0;
		}		
	}
  ```
  
  
  점군서 포인트 선태(위치 좌표로)
  
  ```cpp

pcl::PointXYZRGB searchPoint;
searchPoint.x = 0.026256f;
searchPoint.z = 0.929567f;
  ```
  
  점군서 포인트 선태(인덱스로)
  ```cpp
 pcl::io::loadPCDFile<pcl::PointXYZRGB>("cloud_cluster_0.pcd", *cloud); 
 pcl::PointXYZRGB searchPoint = cloud->points[3000]; //기준점(searchPoint) 설정 방법 #2(3000번째 포인트)
 ```
 
 
 vh
