Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/trusty64"
  config.vm.provider :virtualbox do |v|
    v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    v.customize ["modifyvm", :id, "--memory", 256]
    v.customize ["modifyvm", :id, "--cpus", 1]
  end
  # config.vm.provision :ansible do |ansible|
  #   ansible.verbose = "vv"
  #   ansible.playbook = "playbooks/deploy.yml"
  # end

  config.vm.define "nginx" do |config|
    config.vm.hostname = 'nginx'

    config.vm.network :private_network, ip: "192.168.56.101"
    config.vm.network :forwarded_port, guest: 22, host: 10122, id: "ssh"
    config.vm.network :forwarded_port, guest: 80, host: 18081
    config.vm.network :forwarded_port, guest: 443, host: 18443
  end

  config.vm.define "web" do |config|
    config.vm.hostname = 'web1'

    config.vm.network :private_network, ip: "192.168.56.102"
    config.vm.network :forwarded_port, guest: 22, host: 10222, id: "ssh"
  end

  config.vm.define "db" do |config|
    config.vm.hostname = 'db'

    config.vm.network :private_network, ip: "192.168.56.103"
    config.vm.network :forwarded_port, guest: 22, host: 10322, id: "ssh"
  end

  config.vm.define "jenkins" do |config|
    config.vm.box = "alonsodomin/ubuntu-trusty64-java8"
    config.vm.hostname = 'jenkins'

    config.vm.network :private_network, ip: "192.168.56.104"
    config.vm.network :forwarded_port, guest: 22, host: 10422, id: "ssh"
    config.vm.network :forwarded_port, guest: 8080, host: 18082, id: "jenkins main port"
  end

end
