# OpenCV-OpenWRT-
可以使用OpenCV的linux版本把该功能做到嵌入式设备当中，然而当处理复杂的时候就需要在服务器平台上进行图像处理运算，本文尝试简单实现这一功能    【OpenWRT】 OpenWRT是广泛使用的开源路由器操作系统，因为开源所以强大。  手头有一台MW151路由，据悉该路由与TP-Link 703n的区别只是USB口，改装升级内存和flash，刷上703n的OpenWRT固件，一台适宜开发的设备诞生了。

【步骤：路由】

路由器上的原材料：
703n固件
mjpeg-streamer软件
免驱摄像头（笔者使用的是某宝上淘来的东芝笔记本拆机摄像头）
首先要实现图像传输需要在路由上挂载USB摄像头，本文挂出来的路由固件已经有具有相应组件（kmod-video-core和kmod-video-uvc）。
1.安装mjpeg-streamer讲软件解压，把其中的www目录放到路由器的www目录下，改名为camwww，向路由器上传文件可以用WinSCP，重启路由
  
2.插好摄像头开机，启动mjpeg-streamer，ssh登陆路由（可以用Putty软件ssh登录）输入以下命令
mjpg_streamer -i "input_uvc.so  -d /dev/video0" -o "output_http.so -p 8080 -w /www/camwww"
复制代码

8080指视频流使用端口，可以自定义，其余选项为默认分辨率640x480，30fps
  

用户可以根据自己的需求自定义，例如
mjpg_streamer -i "input_uvc.so -f 15 -r 320*240 -d /dev/video0" -o "output_http.so -p 8080 -w /www/camwww"
复制代码

表示分辨率320x240，15fps

这时候可以看到摄像头的LED被点亮，结束操作时在SSH窗口中按Ctrl+C键可退出mjpg-streamer
  

此时用浏览器（建议用火狐）连接路由器打开以下地址，可以查看摄像头的实时图像

不让发URL，到我的博客<span style="font-family: 'ms shell dlg'; font-size: 14px; line-height: 28px; background-color: rgb(255, 255, 255);">hosea1008.github.io看</span>
复制代码

至此，OpenWRT已经实现挂载摄像头进行无线监控。



【步骤：OpenCV】
电脑是已经安装好Visual Studio 2013并配置好OpenCV 2.4.10

要用OpenCV对路由器传输的实时图像进行处理，只需要让程序从网页获取图像，在while循环里不断把采集的图像转换成Mat矩阵（新版本的OpenCV正在逐步淘汰IplImage结构体）既可，令人欣喜的是，OpenCV里的VideoCapture类本身就能从网页获取图像，因此带来了极大的方便，具体如下：
#include "opencv2/highgui/highgui.hpp"
#include "opencv2/imgproc/imgproc.hpp"
#include <iostream>
#include <stdio.h>
//头文件

using namespace std;
using namespace cv;

/** @function main */
int main(int argc, char** argv)
{
        Mat src;
        cv::VideoCapture vcap;

        const string address = 不让发URL";

        if (!vcap.open(address))
        {
                cout << "Error opening video stream" << endl;
                return -1;
        }

        cout << "Stream opened" << endl;

        while (1)
        {
                
                vcap >> src;
                // your code here
                /// Show your results
                namedWindow("Cam", CV_WINDOW_AUTOSIZE);
                imshow("Cam", src);
                if (waitKey(2) == 27)
                        break;
                 // Press "Esc" to exit
        }
        return 0;
}
复制代码

上文中如果address直接等于浏览器地址栏中输入的地址，将不能获取图像，一个解释是，需要让程序知道视频流是jpeg类型，因此“伪造”了一个jpeg文件名

下图是修改OpenCV官网的一个demo，亲测在路由上获取图像可行（分别从笔记本摄像头和路由器挂载的无线摄像头）



 <转载之处>  http://www.opencv.org.cn/forum.php?mod=viewthread&tid=34141&highlight=%E6%91%84%E5%83%8F%E5%A4%B4
   
