# 처리 시간 계산



```cpp
#include <pcl/common/time.h>
start = pcl::getTime ();
//brute_force.setInputCloud (cloud);
stop = pcl::getTime ();
std::cout << "setting up brute force search: " << (bf_setup = stop - start) << std::endl;
```

