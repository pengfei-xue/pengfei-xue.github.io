## Check the open FD limit for a given process in Linux

Count the entries in /proc/<pid>/fd/
The hard and soft limits applying to the process can be found in `/proc/<pid>/limits`


## ubunut upstart坑

使用upstart 启动服务，ulimit没有正确设置，没有读取limits.conf 以及 ulimit 相关配置。

http://bryanmarty.com/2012/02/10/setting-nofile-limit-upstart/
http://stackoverflow.com/questions/19995855/increase-max-open-files-for-ubuntu-upstart-initctl
