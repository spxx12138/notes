
	- 基于Ubuntu 16.04

1. 在 /etc/init.d 目录下有文件 

# insserv: warning: script '服务名' missing LSB tags and overrides #

脚本没有按照LSB规范编写，要在#!/bin/bash下面添加：
	### BEGIN INIT INFO
	# Provides:          bbzhh.com
	# Required-Start:    $local_fs $network
	# Required-Stop:     $local_fs
	# Default-Start:     2 3 4 5
	# Default-Stop:      0 1 6
	# Short-Description: tomcat service
	# Description:       tomcat service daemon
	### END INIT INFO
	
安装服务时