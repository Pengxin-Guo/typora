### DBMS_project

1. 项目说明

2. **项目上线**

   - 首先，将`master`端做成后台进程（守护进程），即将`master`的所有代码放在`fork`里面；

   - 写`start`脚本和`stop`脚本，放在了`/usr/bin/gpx_pihealth`下面，

     `gpx_pihealthd.start`代码如下：

     ```bash
     #!/bin/bash
     
     if [[ ! -e /etc/gpx_pihealth.pid ]]; then
            touch /etc/gpx_pihealth.pid
     fi       
     
     source /etc/gpx_pihealth.conf
     
     pre_pid=`cat /etc/gpx_pihealth.pid`
     
     if test -n $pre_pid ;then
             ps -ef | grep -w ${pre_pid} | grep master > /dev/null
             if [[ $? == 0 ]]; then
                 echo "Pihealthd has already started."
     	    exit 0
             else
                 echo "Pihealthd is starting."
     	    /home/gpx//Github/DBMS_project/master/master
     	    echo "Pihealthd started."
             fi
     else 
     	    echo "Pihealthd is starting."
     	    /home/gpx/Github/DBMS_project/master/master 
     	    echo "Pihealthd start."
     fi
     ```

     `gpx_pihealthd.stop`代码如下：

     ```bash
     #!/bin/bash
     
     echo "Pihealthd is stoping."
     pid=`cat /etc/gpx_pihealth.pid`
     kill $pid
     echo "Pihealthd stop."
     ```

   - 设置系统`service`，在`/lib/systemd/system`下面写一个`gpx_pihealthy.service `，代码如下：

     ```bash
     [Unit]
     Description=gpx_pihealthd-1.0
     After=syslog.target network.target remote-fs.target nss-lookup.target
     
     [Service]
     Type=forking
     ExecStart=/usr/bin/gpx_pihealth/gpx_pihealthd.start
     ExecStop=/usr/bin/gpx_pihealth/gpx_pihealthd.stop
     
     [Install]
     WantedBy=multi-user.target
     ```

     之后，启动服务代码：`systemctl start gpx_pihealthy.service`，

     ​	查看服务状态代码：`systemctl status gpx_pihealthy.service`，

     ​	停止服务代码：`systemctl stop gpx_pihealthy.service`。

     注：如果修改了`gpx_pihealthy.service`之后，需要重新加载，命令如下`stemctl daemon-reload`。

   - 设置开机自启：`systemctl enable gpx_pihealthy.service`，

     关闭开机自启：`systemctl disable gpx_pihealthy.service`。