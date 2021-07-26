# <center>以 Mapbox Terrian-RGB 模型发布高程数据</center>

## Step 1    安装 Python3

~~~shell
# 下载 Python3 安装包
# 解压缩 .tgz 文件
tar -xvf Python-3.9.6.tgz
# 编译安装 Python3
./configure
make
make install
~~~

## Step 2    安装 conda

~~~shell
# 下载安装包 https://docs.conda.io/en/latest/miniconda.html
# 赋予执行权限
./iniconda3-py39_4.9.2-Linux-x86_64.sh
~~~

## Step 3    安装 gdal

开源的空间数据处理程序

~~~shell
conda install -c conda-forge gdal
~~~

## Step 4    安装 rasterio

MapBox 在 gdal 基础上开发的栅格工具

~~~shell
sudo add-apt-repository ppa:ubuntugis/ppa
sudo apt-get update
sudo apt-get install python-numpy gdal-bin libgdal-dev
pip install rasterio
~~~

## Step 5    安装 rio-rgbify

MapBox 发布的将 dem 栅格编码为 rgb 栅格的 rasterio 插件

~~~shell
pip install rio-rgbify
~~~

## Step6    数据预处理

1. 确定坐标系

   ***GeoTiff 的坐标系必须是 WGS84 Web 墨卡托 (EPSG:3857)。*** 通常 Geo Tiff 格式的灰度栅格图片看起来是黑白的。

   <img src="https://gitee.com/shiayanga/pictures/raw/master/mapbox terrain-rgb/dem.png" style="margin-left:0px;zoom:53%;">

   如果不清楚坐标系的话，可以使用 rasterio 提供的命令行来获取坐标系：

   ```shell
   rio info --indent 2 xxxx.tif
   ```

2. 转换坐标系，清理负数值

   + 如果坐标系不是 `EPSG:3857`，就需要进行坐标转换；

   + 用 **负数值表示无数据** ，而 Terrain-RGB 无法表示负值，所以需要进一步处理；

   ~~~shell
   # 同时进行坐标系转换和清楚表示无数据的负值
   gdalwarp -t_srs EPSG:3857 -dstnodata None -co TILED=YES -co COMPRESS=DEFLATE -co BIGTIFF=IF_NEEDED xxxx.tif xxxx_n.tif
   ~~~

3. 将转换后的文件转换为 Terrain-RGB 格式

   Terrain-RGB 用 3 个 byte 通过 rgb 三通道来表示高程， 比原来的灰度 tiff 要小很多

   + 将 **灰度数据** 转换为 **RGB 数据**

     高度计算公式：

   ~~~text
   height = -10000 + ((R * 256 * 256 + G * 256 + B) * 0.1)
   ~~~


   ​	因此设置 `ribify` 的参数 `base value` 的参数为 `-10000` ， interval 为 `0.1` ，继续输入以下命令：

   ~~~shell
   rio rgbify -b -10000 -i 0.1 xxxx_n.tif xxxx_n_rgb.tif
   ~~~

   <img src="https://gitee.com/shiayanga/pictures/raw/master/mapbox terrain-rgb/rgb.png" style="margin-left:0px;zoom:53%;">

   

   ## Step7    通过 GeoServer 发布切片服务

   1. 创建工作区

      <img src="https://gitee.com/shiayanga/pictures/raw/master/mapbox terrain-rgb/workspace.png" alt="workspace.png" style="zoom:53%;margin-left:0" />

   2. 创建数据存储

      数据源选择 GeoTIFF

      <img src="https://gitee.com/shiayanga/pictures/raw/master/mapbox terrain-rgb/datastore-geotiff.png" style="zoom:53%;margin-left:0px" />

      <img src="https://gitee.com/shiayanga/pictures/raw/master/mapbox terrain-rgb/datastore-geotiff2.png" style="zoom:53%;margin-left:0px" />

   3. 发布图层

      <img src="https://gitee.com/shiayanga/pictures/raw/master/mapbox terrain-rgb/layer.png" style="zoom:53%;margin-left:0" />

   4. 创建图层组

      <img src="https://gitee.com/shiayanga/pictures/raw/master/mapbox terrain-rgb/layergroup.png" style="zoom:50%;margin-left:0" />

   ## Step 8    查看结果

   ## ![](https://gitee.com/shiayanga/pictures/raw/master/mapbox terrain-rgb/result1.png)

   ## ![](https://gitee.com/shiayanga/pictures/raw/master/mapbox terrain-rgb/result2.png)

