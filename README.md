# 服务器安装centos并配置网络
（整体流程参考https://www.osyunwei.com/archives/7829.html）

1. 制作centos的u盘启动，注意安装时盘符选择问题。在安装时tab后修改对应位置为自己的u盘盘符名。
 	- 第一种解决方案（参考https://blog.csdn.net/Bruce_You/article/details/75332090），找到自己u盘的盘符名。
即修改第二步中按TAB键出来的命令：将命令修改为：>vmlinuz initrd=initrd.img linux dd quiet
	- 第二种解决方案：直接将u盘名称改为例如CENTOS7,然后把label修改为CENTOS7即可

2. 安装、挂载硬盘（系统默认分区）

3. 网络配置
	- 打开配置文件/etc/sysconfig/network-scripts/xxx（xxx为对应的网线端口名称），添加修改信息：
		- HWADDR=00:0C:29:8D:24:73（网卡的mac地址，通过ip addr查看）
		- BOOTPROTO=static  #启用静态IP地址
		- ONBOOT=yes  #开启自动启用网络连接
		- IPADDR=192.168.21.128  #设置IP地址
		- PREFIX=24  #设置子网掩码
		- GATEWAY=192.168.21.2  #设置网关
		- DNS1=  #设置主DNS

4. ssh（22端口centos默认打开，无需配置）

5. 路由器端口配置（远程访问，添加虚拟服务器转发规则）

6. gpu驱动等安装

	- 参考（https://blog.csdn.net/xueshengke/article/details/78134991、https://blog.csdn.net/u012325865/article/details/73034018）
	1. 下载nvidia显卡驱动和cuda，根据自己的型号官网下载即可
	2. 安装gcc等（yum install gcc 、yum install gcc-c++、yum install kernel-devel、yum install kernel-headers）
	3. 赋予权限：chmod 755 NVIDIA-Linux-x86_64-390.77.run ,chmod 755 cuda_9.2.148_396.37_linux
	4. 屏蔽nouveau驱动：
		1. 将blacklist nouveau
options nouveau modeset=0 加入到/etc/modprobe.d/nvidia-installer-disable-nouveau.conf和/lib/modprobe.d/nvidia-installer-disable-nouveau.conf中
		2. 重做 initramfs 镜像：
	 cp /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r).img.bak
    dracut /boot/initramfs-$(uname -r).img $(uname -r)
    rm /boot/initramfs-$(uname -r).img.bak ; 这一步可不执行
	5. 重启
	6. 安装nvidia驱动：
		- ./NVIDIA-Linux-x86_64-390.77.run --kernel-source-path=/usr/src/kernels/3.10.0-862.9.1.el7.x86_64  -k $(uname -r)   此处/kernels后信息需要查看自己内核版本uname -r
		- 安装成功后 nvidia-smi可看到驱动信息
	7. 安装cuda：./cuda_9.2.148_396.37_linux --kernel-source-path=/usr/src/kernels/3.10.0-862.9.1.el7.x86_64      此处/kernels后信息需要查看自己内核版本uname -r
	8. 配置环境变量：vim /etc/profile
export PATH=/usr/local/cuda/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH
	9. source /etc/profile ; 使环境变量立即生效
	10. 安装成功后 nvcc --version检查即可
