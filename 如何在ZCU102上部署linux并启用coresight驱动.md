# 如何在ZCU102上部署linux并启用coresight驱动
版本|时间|修订内容
|:-:|:-:|:-:|
v1.0|2022.04.11|记录部署过程
## 一、部署环境、软件、工具
原料|版本|备注
|:-:|:-:|:-:|
linux|5.12.0-rc1|5.15版本存在内核无法识别文件系统所在的SD卡分区2的情况，5.16及以上版本的编译需要libyaml库作为依赖，不清楚petalinux如何维护编译内核源码的环境，故放弃。
ZCU102|rev1.1|目前大部分资料都是针对rev1.0的，是否存在区别尚不明确
petalinux|2020.2|目前最新版本为2021.2
BSP|xilinx-zcu102-v2020.2-final|目前最新版本为2021.2
## 二、部署流程
1. 下载`linux-xlnx`<br/>
```
$git clone https://github.com/Xilinx/linux-xlnx.git
$git checkout zynqmp-soc-for-v5.13
```
2. `linux-xlnx`修改配置<br/>

修改`linux-xlnx/drivers/hwtracing/Kconfig`，加入行
```
source "drivers/hwtracing/coresight/Kconfig"
```
在`linux-xlnx`的主分支找到`linux-xlnx/arch/amd64/configs/xilinx_defconfig`并将其拷贝至当前`linux-xlnx`相同目录下

3. 建立petalinux项目
```
petalinux-create -t project -s xilinx-zcu102-v2020.2-final.bsp
```
4. 构建设备树

修改

5. petalinux项目配置

6. 内核配置

7. petalinux项目构建并打包

8. 写入SD卡

