官网能做到的就是让你开个机。。

涉及到网络部分的配置就GG：
http://stackoverflow.com/questions/38636023/vagrant-not-supported-the-capability-change-host-name

我要开多个VM啊：
http://www.thisprogrammingthing.com/2015/multiple-vagrant-vms-in-one-vagrantfile/

vagrant 配置
config.vm.define "nginx" do |config|
    config.vm.box = "ubuntu/trusty64"
    config.vm.hostname = 'nginx'

    config.vm.network :private_network, ip: "192.168.56.101"
    config.vm.network :forwarded_port, guest: 22, host: 10122, id: "ssh"
    config.vm.network :forwarded_port, guest: 80, host: 8080
    config.vm.network :forwarded_port, guest: 443, host: 8443

    config.vm.provider :virtualbox do |v|
      v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
      v.customize ["modifyvm", :id, "--memory", 512]
      v.customize ["modifyvm", :id, "--name", "nginx"]
    end
  end

  config.vm.define "web1" do |config|
    config.vm.box = "ubuntu/trusty64"
    config.vm.hostname = 'web1'

    config.vm.network :private_network, ip: "192.168.56.102"
    config.vm.network :forwarded_port, guest: 22, host: 10222, id: "ssh"
  end

  config.vm.define "web2" do |config|
    config.vm.box = "ubuntu/trusty64"
    config.vm.hostname = 'web2'

    config.vm.network :private_network, ip: "192.168.56.103"
    config.vm.network :forwarded_port, guest: 22, host: 10322, id: "ssh"
  end

  config.vm.define "db" do |config|
    config.vm.box = "ubuntu/trusty64"
    config.vm.hostname = 'db'

    config.vm.network :private_network, ip: "192.168.56.104"
    config.vm.network :forwarded_port, guest: 22, host: 10422, id: "ssh"
  end
与ansible配合
现在我用ansible只是把所有需要的依赖工具安装好

配置默认信息
cd playbooks
sudo vim ansible.cfg
[defaults]
hostfile=hosts
host_key_checking = False

配置登录信息
cd nginx-workshop
mkdir playbooks
vim hosts
[nginx]
nginx ansible_ssh_host=127.0.0.1 ansible_connection=ssh ansible_ssh_user=vagrant ansible_ssh_port=10122 ansible_ssh_private_key_file=../.vagrant/machines/nginx/virtualbox/private_key
[web]
web1 ansible_ssh_host=127.0.0.1 ansible_connection=ssh ansible_ssh_user=vagrant ansible_ssh_port=10222 ansible_ssh_private_key_file=../.vagrant/machines/web1/virtualbox/private_key
web2 ansible_ssh_host=127.0.0.1 ansible_connection=ssh ansible_ssh_user=vagrant ansible_ssh_port=10322 ansible_ssh_private_key_file=../.vagrant/machines/web2/virtualbox/private_key
[db]
db ansible_ssh_host=127.0.0.1 ansible_connection=ssh ansible_ssh_user=vagrant ansible_ssh_port=10422 ansible_ssh_private_key_file=../.vagrant/machines/db/virtualbox/private_key

测试是否可连
ansible nginx -i hosts -m ping

这样配置之后在本机上可以手动连接
web.vm.network :forwarded_port, guest: 22, host: 10122, id: "ssh"
ssh vagrant@127.0.0.1 -p 10122

看web跑了多久
ansible web -m command -a uptime

SSH
把主机的公钥发给客户机让主机登录
scp id_rsa.pub vagrant@192.168.56.101:/home/vagrant/
cat id_rsa.pub >> authorized_keys
ansible使用的是私钥登录，-i加上identify file，就是私钥，用私钥当时创建人的身份登录，如果私钥对应的公钥在vagrant授权文件中
ssh vagrant@127.0.0.1 -p 10122 -i /Users/tywang/WuhanWork/nginx-workshop/.vagrant/machines/nginx/virtualbox/private_key

安装nginx（手动）
ansible web -s -m apt -a "name=nginx update_cache=yes"
开启服务检测服务
ansible web -s -a "service nginx start"
ansible web -s -m service -a "name=nginx state=started"
证书？？？
cd playbooks
openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -subj /CN=localhost -keyout nginx.key -out nginx.crt
安装nginx git bundle npm
➜  playbooks git:(master) ✗ ansible-playbook ../deploy.yml
（自动）
├── Vagrantfile
├── deploy.yml
└── playbooks
    ├── ansible.cfg
    ├── hosts
    ├── install_nginx.yml
# ./nginx-workshop/deploy.yml

- hosts: nginx
  tasks:
    - include: 'playbooks/install_nginx.yml'
      become: yes

- hosts: web
  tasks:
    - include: 'playbooks/install_web.yml'
      become: yes

- hosts: db
  tasks:
    - include: 'playbooks/install_db.yml'
      become: yes

# ./nginx-workshop/tasks/install_nginx.yml

- name: NGINX | Installing NGINX
  apt:
    name: nginx
    state: latest

- name: NGINX | Starting NGINX
  service:
    name: nginx
    state: started
# ./nginx-workshop/tasks/install_web.yml

- name: GIT | Installing GIT
  apt:
    name: git
    state: latest

- name: RUBY | Installing RUBY
  apt:
    name: ruby-full
    state: latest

- name: BUNDLE | Installing BUNDLE
  command: bash -lc "gem install bundler"

#gem:
   #name: bundler(gem install --include-dependencies --user-install --no-rdoc --no-ri 'install bundler' 这样写就GG了,因为user-install这个参数将bundler装在了~/.gem/ruby/1.9.1下，装完之后会执行加入环境变量的脚本，但这个path不在环境变量中无法执行，如果模拟登录之后执行gem install bundler就不会带上user-install参数)

# ./nginx-workshop/tasks/install_db.yml

- name: MONGODB | Installing MONGODB
  apt:
    name: mongodb-server
    state: latest

- name: MONGODB | Start MongoDB Server
  command: bash -lc "mongod --fork --logpath '/dev/null'"

强制关闭后export LC_ALL=C或者/data/db rm .lock
和go配合
go agent与web/nginx的免密码登录
go agent上：
ssh-keygen
scp ~/.ssh/id_rsa.pub vagrant@192.168.56.102:~
web上：
cat ~/id_rsa.pub >> ~/.ssh/authorized_keys

todo:
db上后台重启
AWS
