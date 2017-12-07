#这是什么
用vagrant开启jenkins nginx web db虚拟机并安装对应环境

#测试
测试是否可连
`ansible nginx -i hosts -m ping`

这样配置之后在本机上可以手动连接
```
web.vm.network :forwarded_port, guest: 22, host: 10122, id: "ssh"
ssh vagrant@127.0.0.1 -p 10122
```

看web跑了多久
`ansible web -m command -a uptime`

把主机的公钥发给客户机让主机登录
```
scp id_rsa.pub vagrant@192.168.56.101:/home/vagrant/
cat id_rsa.pub >> authorized_keys
ansible使用的是私钥登录，-i加上identify file，就是私钥，用私钥当时创建人的身份登录，如果私钥对应的公钥在vagrant授权文件中
ssh vagrant@127.0.0.1 -p 10122 -i /Users/tywang/WuhanWork/nginx-workshop/.vagrant/machines/nginx/virtualbox/private_key
```

安装nginx（手动）
`ansible web -s -m apt -a "name=nginx update_cache=yes"`

开启服务检测服务
```
ansible web -s -a "service nginx start"
ansible web -s -m service -a "name=nginx state=started"
```

证书
`openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -subj /CN=localhost -keyout nginx.key -out nginx.crt`

SSH免密码登录
```
sudo su jenkins
ssh-keygen
ssh-copy-id vagrant@192.168.56.102
```

#参考
涉及到网络部分的配置就GG：
http://stackoverflow.com/questions/38636023/vagrant-not-supported-the-capability-change-host-name

开多个VM：
http://www.thisprogrammingthing.com/2015/multiple-vagrant-vms-in-one-vagrantfile/
