# AssemblyLineRobot-Coppeliasim
### 测试结果
首先，通过传送带末端的检测传感器，物料在传送至传送带末端目标位置时，传感器识别到物体并马上将信号传输给传送带和JAKA12，传送带随即暂停传送，直至JAKA12机械臂进行物料抓取。另外，根据传感器的信号，计数器也将记录已经传送的物料数目，并用UI组件进行数量显示。

然后，根据传送带侧方的摄像头及图像处理算法识别物料的颜色。物料的识别结果同步到JAKA12机械臂，若为紫色物料，则通过机械臂放置到地面上的目标位置；若为橙色物料，则先用机械臂“井”型堆叠放置在youBot物料小车上。

下一步，当视觉图像中没有需要传送的物料时，物料小车通过轨迹跟随算法到达目标物料池前方，紧接着根据物料池前方安装的激光测距传感器检测物料小车与物料池的距离，实现小车精准靠近物料池并当距离小于阈值时停止运动。

最后，物料小车先将小车上原本有的浅蓝色方形物料放到物料池侧边平台上，其它物料由youBot上的机械臂推送到物料池中，整个流水线仿真场景结束。

整个协作流水线机器人通过仿真验证，能够较好的完成当前任务，但是在youBot机械臂推送到物料池的过程中存在物料散落的现象。


![在这里插入图片描述](https://img-blog.csdnimg.cn/5b57ad62232f47e8880ec70135eb8ac9.gif)

### 视觉处理算法
视觉处理的摄像头采用了Vision sensor，其得到的图像可以关联到Floating view，从而实时进行观测。图像处理采用了simVision库函数，其是用于执行简单图像处理和处理特殊类型视觉算法的API函数，足够满足处理本项目的图像。

 1. Step1：将摄像头获得的图像句柄用simVision.sensorImgToWorkImg()转移到工作空间，从而对图像进行进一步操作。
 2. Step2：通过simVision.selectiveColorOnWorkImg() 将颜色阈值内的紫色物料分割出来，并且用simVision.blobDetectionOnWorkImg() 将物料识别出来，阈值外的颜色存储到Buffer1中。
 3. Step3：用simVision.workImgToBuffer2() 函数将工作空间图像转移到Buffer2中，再用 simVision.swapWorkImgWithBuffer1() 函数将工作空间和Buffer1中的图像互换，重复Step2将橙色物料识别出来。
 4. Step4：使用simVision.swapBuffers()将Buffer中的图像互换，最后用simVision.addBuffer1ToWorkImg()函数将Buffer1中的图像加到工作空间图像上，得到最终图像处理的结果。![在这里插入图片描述](https://img-blog.csdnimg.cn/dc3b2313a7134e72ba140cb1e0f38d58.png#pic_center)
 5. Setp5：将得到的物料位置和颜色信息显示在辅助控制台窗口上并通过sim.writeCustomDataBlock()将上述信息存储起来，供其它协作机器人获取，实现机器人之间信息的传递。![在这里插入图片描述](https://img-blog.csdnimg.cn/b59902ecf3a549ddb540754cf9cf00f4.png#pic_center)
 ### 基于Dummy和Path的轨迹规划   
在Coppeliasim中，Path是Scence的一种“实体”，与model一样，用户可以使用Path来自定义各种运动路径，简单来说，从Path上我们可以给定某一个时刻机器人运动到某一点的位置以及其姿态，定义机器人在整个路径中每一时刻的运动过程，Coppliasim中默认的Path是通过给定一些关键的控制点，并在这些控制点中间采用贝塞尔曲线插值来得到光滑路径。Dummy常用来与Path相结合，Dummy可以沿着Path向前运动，实时获取当前时刻在Path上的位置和姿态，作为路径上的一个“领航者”。

 1. Step1：获得Dummy和Path的句柄。
 2. Step2：获得视觉传感器DataBlock上存储的物料信息，若传送带上无物料，则调用sim.followPath()函数，使得Dummy顺着Path的轨迹以一定的速度运动。
 3. Step3：youBot获得Dummy句柄，并得到Dummy的位姿信息。
 4. Step4：youBot用PD算法跟随Dummy位姿进行运动，实现轨迹的跟随。
![在这里插入图片描述](https://img-blog.csdnimg.cn/de3ff1a51dc643cba8f5e92e34aa9a3a.png)
