- [1、GIS坐标系转换](#1gis坐标系转换)
- [2、Tomcat跨域](#2tomcat跨域)
  - [问题描述](#问题描述)
  - [解决方法](#解决方法)
- [3、GeoServer Rest API访问鉴权失败](#3geoserver-rest-api访问鉴权失败)
  - [问题描述](#问题描述-1)
  - [解决方法](#解决方法-1)
- [4、GeoServer跨域问题](#4geoserver跨域问题)
  - [思路](#思路)
- [5、查询图层组有结果，但是加载时404](#5查询图层组有结果但是加载时404)
- [6、加载图层组时控制台有大量报错或图层组为空](#6加载图层组时控制台有大量报错或图层组为空)
- [7、使用GeoServer查看图层组有结果，但是加载时404](#7使用geoserver查看图层组有结果但是加载时404)

# 1、GIS坐标系转换
1. EPSG:4326
~~~ text
WGS84 是目前最流行的地理坐标系统。在国际上，每个坐标系统都会被分配一个 EPSG 代码，EPSG:4326 就是 WGS84 的代码。

GPS是基于WGS84的，所以通常我们得到的坐标数据都是WGS84的。一般我们在存储数据时，仍然按WGS84存储。
~~~

2. EPSG:3857
~~~text
伪墨卡托投影，也被称为球体墨卡托，Web Mercator。它是基于墨卡托投影的，把 WGS84坐标系投影到正方形。

我们前面已经知道 WGS84 是基于椭球体的，但是伪墨卡托投影把坐标投影到球体上，这导致两极的失真变大，但是却更容易计算。这也许是为什么被称为”伪“墨卡托吧。
另外，伪墨卡托投影还切掉了南北85.051129°纬度以上的地区，以保证整个投影是正方形的。

因为墨卡托投影等正形性的特点，在不同层级的图层上物体的形状保持不变，一个正方形可以不断被划分为更多更小的正方形以显示更清晰的细节。
很明显，伪墨卡托坐标系是非常显示数据，但是不适合存储数据的，通常我们使用WGS84 存储数据，使用伪墨卡托显示数据。

Web Mercator 最早是由 Google 提出的，当前已经成为 Web Map 的事实标准。但是也许是由于上面”伪“的原因，最初 Web Mercator 被拒绝分配EPSG 代码。于是大家普遍使用 EPSG:900913（Google的数字变形） 的非官方代码来代表它。
直到2008年，才被分配了EPSG:3785的代码，但在同一年没多久，又被弃用，重新分配了 EPSG:3857 的正式代码，使用至今。
~~~

3. EPSG:3857转换经纬度(EPSG:4326)
~~~javascript
// javascript 转换
function mercatorTolonlat(mercator){
    var lonlat={x:0,y:0};
    var x = mercator.x/20037508.34*180;
    var y = mercator.y/20037508.34*180;
    y= 180/Math.PI*(2*Math.atan(Math.exp(y*Math.PI/180))-Math.PI/2);
    lonlat.x = x;
    lonlat.y = y;
    return lonlat;
}
~~~

4. EPSG:3857转换经纬度(EPSG:4326)转换工具

<script type="text/javascript" src="http://libs.baidu.com/jquery/2.0.0/jquery.min.js"></script>
<div>
    X坐标：<input type="text" id="x" placeholder="输入X坐标">
    <br>
    Y坐标：<input type="text" id="y" placeholder="输入X坐标">   
    <button style="margin-left:5px" onclick="mercatorTolonlat()">转换</button>
    <br>
    经纬度：<input style="width:300px" placeholder="结果将在这里输出" id="result4326"/>
</div>

<script type="text/javascript">
    function mercatorTolonlat() {
        let coorx = $("#x").val();
        let coory = $("#y").val();
        if($.trim(coorx) == "" || $.trim(coory) == ""){
            alert("有坐标未输入~");
            return;
        }
        var lonlat = { x: 0, y: 0 };
        var x = coorx / 20037508.34 * 180;
        var y = coory / 20037508.34 * 180;
        y = 180 / Math.PI * (2 * Math.atan(Math.exp(y * Math.PI / 180)) - Math.PI / 2);
        lonlat.x = x;
        lonlat.y = y;
        $("#result4326").val(x + "," + y);
    }
</script>
5. 经纬度(EPSG:4326)转换EPSG:3857
~~~javascript
// javascript 转换
function lonLat2Mercator(lonlat){
    var mercator = {};
    var earthRad = 6378137.0;
    mercator.x = lonlat.lng * Math.PI / 180 * earthRad;
    var a = lonlat.lat * Math.PI / 180;
    mercator.y = earthRad / 2 * Math.log((1.0 + Math.sin(a)) / (1.0 - Math.sin(a)));
    return mercator;
}
~~~

1. 经纬度(EPSG:4326)转换EPSG:3857转换工具

<script type="text/javascript" src="http://libs.baidu.com/jquery/2.0.0/jquery.min.js"></script>
<div>
    经度：<input type="text" id="lon" placeholder="输入经度">
    <br>
    纬度：<input type="text" id="lat" placeholder="输入纬度">   
    <button style="margin-left:5px" onclick="lonLat2Mercator()">转换</button>
    <br>
    3857坐标：<input style="width:300px" placeholder="结果将在这里输出" id="result3857"/>
</div>

<script type="text/javascript">
    function lonLat2Mercator() {
        let coorx = $("#lon").val();
        let coory = $("#lat").val();
        // if($.trim(coorx) == "" || $.trim(coory) == ""){
        //     alert("有坐标未输入~");
        //     return;
        // }
        // let x = coorx * 20037508.34 / 180;
        // let y = Math.log(Math.tan(((90 + coory) * Math.PI) / 360)) / (Math.PI / 180);
        // y = y * y / 180;
        // $("#result3857").val(x + "," + y);
        var mercator = {};
        var earthRad = 6378137.0;
        // console.log("mercator-poi",poi);
        mercator.x = coorx * Math.PI / 180 * earthRad;
        var a = coory * Math.PI / 180;
        mercator.y = earthRad / 2 * Math.log((1.0 + Math.sin(a)) / (1.0 - Math.sin(a)));
        // console.log("mercator",mercator);
        $("#result3857").val(mercator.x + "," + mercator.y);
    }
</script>

# 2、Tomcat跨域
## 问题描述
    浏览器访问Tomcat时会出现跨域问题

## 解决方法
    可以通过配置Tomcat跨域过滤器解决跨域问题

修改Tomcat conf目录下web.xml，在Default Session Configuration一行前面添加
~~~xml
<filter>
    <filter-name>CorsFilter</filter-name>
    <filter-class>org.apache.catalina.filters.CorsFilter</filter-class>
    <init-param>
        <param-name>cors.allowed.origins</param-name>
        <param-value>*</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>CorsFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
~~~
Tomcat CORS跨域支持也可以参考https://tomcat.apache.org/tomcat-9.0-doc/config/filter.html

# 3、GeoServer Rest API访问鉴权失败
## 问题描述
    前端在调用geoserver rest api时，会出现401鉴权失败的问题。
    查看geoserver的文档也没有看到传认证信息参数的地方。

## 解决方法
修改rest.properties文件，允许匿名访问(临时解决方法，安全性低)
~~~text
文件位置：
geoserver/data/security

修改内容：
    /**;POST,GET,PUT=IS_AUTHENTICATED_ANONYMOUSLY
    /**;DELETE=ROLE_ADMINISTRATOR
~~~
# 4、GeoServer跨域问题

## 思路
~~~text
    GeoServer自己带有解决跨域的方法：默认在geoserver/WEB-INF/web.xml中是注释状态的。
~~~

大概在162行到183行这里，有以下内容：

~~~xml
<filter>
    <filter-name>cross-origin</filter-name>
    <filter-class>org.eclipse.jetty.servlets.CrossOriginFilter</filter-class>
    <init-param>
        <param-name>chainPreflight</param-name>
        <param-value>false</param-value>
    </init-param>
    <init-param>
        <param-name>allowedOrigins</param-name>
        <param-value>*</param-value>
    </init-param>
    <init-param>
        <param-name>allowedMethods</param-name>
        <param-value>GET,POST,PUT,DELETE,HEAD,OPTIONS</param-value>
    </init-param>
    <init-param>
        <param-name>allowedHeaders</param-name>
        <param-value>*</param-value>
    </init-param>
</filter>
~~~
193-198行有以下内容：
~~~xml
<filter-mapping>
    <filter-name>cross-origin</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
~~~

~~~text
    修改完文件后，将 jetty-servlets-9.4.12.v20180830.jar 和 jetty-util-9.4.12.v20180830.jar两个jar包放入/geoserver/WEB-INF/lib中，重启Tomcat
~~~


# 5、查询图层组有结果，但是加载时404
~~~text
    将查询到的图层组所在的工作区设置为默认工作区
~~~

# 6、加载图层组时控制台有大量报错或图层组为空
~~~text
    1、打开geoserver，找到该图层组所包含的图层；
    2、选择 “重新载入要素类型” 
    3、在 “边框” 选项中点击 “从数据中计算” 和 “Compute from native bounds” ；
    4、保存
~~~

# 7、使用GeoServer查看图层组有结果，但是加载时404
~~~text
     查看图层组所在的工作区是否为默认工作区
~~~













