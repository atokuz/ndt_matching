# 3D NDT Matching on ROS2

Instructions:

```
source /opt/ros/crystal/setup.bash
git clone https://github.com/atokuz/ndt_matching.git
cd ndt_matching/
colcon build
source install/local_setup.bash
ros2 run ndt_matching ndt_node
```


I needed to make some changes to CMakeLists.txt file.

PCL Library added
```
set(PCL_INCLUDE_DIRS /usr/include/pcl-1.8) 

target_include_directories(ndt_lib PUBLIC
  ${PCL_INCLUDE_DIRS}
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>)

target_include_directories(ndt_node PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>)

target_link_libraries(ndt_node ndt_lib ${PCL_LIBRARIES})
```
This was also missing in CMakeLists.txt
   ```
ament_target_dependencies(
  ndt_lib
  "rclcpp"
)
```
Extra files added for the library
   ```
set(srcs
        src/ndt_lib.cpp
        src/VoxelGrid.cpp
        src/Octree.cpp
        )

set(incs
        include/ndt_matching/debug.h
        include/ndt_matching/ndt_lib.hpp
        include/ndt_matching/SymmetricEigenSolver.h
        include/ndt_matching/VoxelGrid.h
        include/ndt_matching/Octree.h
        )

add_library(ndt_lib ${incs} ${srcs})
```
I put some symlinks to /usr/local/include just for convenience
```
akif@akif-Aspire-5750:/usr/local/include$ ls -al
total 8
drwxr-xr-x  2 root root 4096 Jan 23 23:07 .
drwxr-xr-x 10 root root 4096 Jul 25  2018 ..
lrwxrwxrwx  1 root root   25 Jan 23 14:29 Eigen -> /usr/include/eigen3/Eigen
lrwxrwxrwx  1 root root   24 Jan 23 03:24 pcl -> /usr/include/pcl-1.8/pcl
lrwxrwxrwx  1 root root   31 Jan 23 23:07 unsupported -> /usr/include/eigen3/unsupported
```
To convert from PointCloud2 to PclCloud I was planning to use pcl_conversions.
But there are still some unresolved [problems](https://github.com/ros2/pcl_conversions/issues/3) with it.
So instead I needed to use PointCloudIterator and copy points one by one.


Most of the code has been adapted from PCL library.
Also some parts of Autoware Lidar localizer code has been used.


