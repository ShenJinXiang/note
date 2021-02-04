# linux防火墙

```
#查看对外开放的端口状态
# 查询已开放的端口
netstat -anp

# 查询指定端口是否已开 
firewall-cmd --query-port=443/tcp
# 提示 yes，表示开启；no表示未开启。


# 查看防火墙状态 
systemctl status firewalld
# 开启防火墙 
systemctl start firewalld  
# 关闭防火墙 
systemctl stop firewalld
# 开启防火墙 
service firewalld start 
# 若遇到无法开启
#先用：
systemctl unmask firewalld.service 
# 然后：
systemctl start firewalld.service

# 添加指定需要开放的端口：
firewall-cmd --add-port=123/tcp --permanent
# 重载入添加的端口：
firewall-cmd --reload


# 没有 iptables

systemctl stop firewalld #关闭防火墙
yum install iptables-services #安装或更新服务
# 再使用
systemctl enable iptables # 启动iptables

systemctl start iptables #打开iptables

#再执行
service iptables save

# 重启iptables服务：
service iptables restart
```

