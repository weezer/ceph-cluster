Vagrant.configure("2") do |config|
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = false
  config.hostmanager.manage_guest = true
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = true
  config.ssh.insert_key = false
  insecure_key_path = File.join(Dir.home, ".vagrant.d", "insecure_private_key")
  insecure_private_key = IO.read(insecure_key_path)

  config.vm.define "chef-server" do |server|
    server.vm.box = "bento/ubuntu-16.04"
    server.vm.hostname = 'chef-server'

    server.vm.network :private_network, ip: "192.168.100.102"

    server.vm.provider :virtualbox do |v|
      v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
      v.customize ["modifyvm", :id, "--memory", 4096]
      v.customize ["modifyvm", :id, "--name", "chef-server"]
      v.cpus = 4
    end

    server.vm.provision "shell", inline: <<-EOF
      cp -r /home/vagrant/.ssh ~/.ssh
      echo '#{insecure_private_key}' > ~/.ssh/id_rsa
      chmod 600 ~/.ssh/id_rsa
    EOF
    server.vm.provision "shell", path: "install-chef-server.sh"

  end

  (1..3).each do |i|
    config.vm.define "chef-node-#{i}" do |node|
      node.vm.box = "bento/ubuntu-16.04"
      node.vm.hostname = "chef-node-#{i}"

      node.vm.network :private_network, ip: "192.168.100.#{103+i}"

      node.vm.provider :virtualbox do |v|
        v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        v.customize ["modifyvm", :id, "--memory", 1024]
        v.customize ["modifyvm", :id, "--name", "chef-node-#{i}"]
      end

      node.vm.provision "shell", inline: <<-EOF
        cp -r /home/vagrant/.ssh ~/.ssh
      EOF
    end
  end

  config.vm.define "chef-workstation" do |workstation|
    workstation.vm.box = "bento/ubuntu-16.04"
    workstation.vm.hostname = 'chef-workstation'
    workstation.vm.network :private_network, ip: "192.168.100.101"
    workstation.vm.provision "file", source: "./knife.rb", destination: "/tmp/knife.rb"
    workstation.vm.provider :virtualbox do |v|
      v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
      v.customize ["modifyvm", :id, "--memory", 1024]
      v.customize ["modifyvm", :id, "--name", "chef-workstation"]
      v.cpus = 2
    end

    workstation.vm.provision "shell", inline: <<-EOF
      mkdir -p ~/learn-chef/.chef
      mv /tmp/knife.rb ~/learn-chef/.chef/
      cp -r /home/vagrant/.ssh ~/.ssh
      echo '#{insecure_private_key}' > ~/.ssh/id_rsa
      chmod 600 ~/.ssh/id_rsa
      apt-get update
      apt-get -y install curl
      curl https://omnitruck.chef.io/install.sh | sudo bash -s -- -P chef-workstation -c stable -v 0.2.41
      scp -o StrictHostKeyChecking=no root@chef-server:/drop/chefadmin.pem ~/learn-chef/.chef/chefadmin.pem
      cd ~/learn-chef/
      knife ssl fetch
      knife ssl check
      for VAR in {1..3}
      do
        knife bootstrap chef-node-"${VAR}" -N node-"${VAR}"
      done
    EOF
  end
end
