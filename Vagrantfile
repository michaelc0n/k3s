default_box = 'ubuntu/bionic64'

Vagrant.configure(2) do |config|
  config.vm.define 'master' do |master|
    master.vm.box = default_box
    master.vm.hostname = "master"
    master.vm.synced_folder ".", "/vagrant", type:"virtualbox"
    master.vm.network 'private_network', ip: "192.168.0.200",  virtualbox__intnet: true
    master.vm.network "forwarded_port", guest: 22, host: 2222, id: "ssh", disabled: true
    master.vm.network "forwarded_port", guest: 22, host: 2000
    master.vm.network "forwarded_port", guest: 6443, host: 6443
    master.vm.network "forwarded_port", guest: 8443, host: 8443
    for p in 30000..30100 # PORTS DEFINED FOR K8S TYPE-NODE-PORT ACCESS
      master.vm.network "forwarded_port", guest: p, host: p, protocol: "tcp"
      end
    master.vm.provider "virtualbox" do |v|
      v.memory = "1024"
      v.name = "master"
      end

    master.vm.provision "shell", inline: <<-SHELL
      sudo apt update
      sudo apt install etcd-server etcd-client -y
      IPADDR=$(ip a show enp0s8 | grep "inet " | awk '{print $2}' | cut -d / -f1)
      curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--node-ip=${IPADDR} --flannel-iface=enp0s8 --write-kubeconfig-mode 644 --kube-apiserver-arg="service-node-port-range=30000-30100" --no-deploy=servicelb --no-deploy=traefik" K3S_STORAGE_BACKEND=etcd3 K3S_STORAGE_ENDPOINT="http://127.0.0.1:2379" sh -
      hostnamectl set-hostname master
      NODE_TOKEN="/var/lib/rancher/k3s/server/node-token"
      while [ ! -e ${NODE_TOKEN} ]
      do
          sleep 1
      done
      cat ${NODE_TOKEN}
      cp ${NODE_TOKEN} /vagrant/
      sudo sed -i 's/ChallengeResponseAuthentication no/ChallengeResponseAuthentication yes/g' /etc/ssh/sshd_config
      sudo systemctl reload ssh
      KUBE_CONFIG="/etc/rancher/k3s/k3s.yaml"
      cp ${KUBE_CONFIG} /vagrant/ #copy contents of "k3s.yaml" to ".kube/config" to 'kubectl' from local-machine
    SHELL
  end

  config.vm.define 'node1' do |node1|
    node1.vm.box = default_box
    node1.vm.hostname = "node1"
    node1.vm.synced_folder ".", "/vagrant", type:"virtualbox"
    node1.vm.network 'private_network', ip: "192.168.0.201",  virtualbox__intnet: true
    node1.vm.network "forwarded_port", guest: 22, host: 2222, id: "ssh", disabled: true
    node1.vm.network "forwarded_port", guest: 22, host: 2001
    node1.vm.provider "virtualbox" do |v|
      v.memory = "1024"
      v.name = "node1"
      end
    
    node1.vm.provision "shell", inline: <<-SHELL
      IPADDR=$(ip a show enp0s8 | grep "inet " | awk '{print $2}' | cut -d / -f1)
      curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--node-ip=${IPADDR} --flannel-iface=enp0s8" K3S_URL=https://192.168.0.200:6443 K3S_TOKEN=$(cat /vagrant/node-token) sh -
      sudo sed -i 's/ChallengeResponseAuthentication no/ChallengeResponseAuthentication yes/g' /etc/ssh/sshd_config
      sudo systemctl reload ssh
    SHELL
  end

  config.vm.define 'node2' do |node2|
    node2.vm.box = default_box
    node2.vm.hostname = "node2"
    node2.vm.synced_folder ".", "/vagrant", type:"virtualbox"
    node2.vm.network 'private_network', ip: "192.168.0.202",  virtualbox__intnet: true
    node2.vm.network "forwarded_port", guest: 22, host: 2222, id: "ssh", disabled: true
    node2.vm.network "forwarded_port", guest: 22, host: 2002
    node2.vm.provider "virtualbox" do |v|
      v.memory = "1024"
      v.name = "node2"
      end
    
    node2.vm.provision "shell", inline: <<-SHELL
      IPADDR=$(ip a show enp0s8 | grep "inet " | awk '{print $2}' | cut -d / -f1)
      curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--node-ip=${IPADDR} --flannel-iface=enp0s8" K3S_URL=https://192.168.0.200:6443 K3S_TOKEN=$(cat /vagrant/node-token) sh -
      rm -f /vagrant/node-token
      sudo sed -i 's/ChallengeResponseAuthentication no/ChallengeResponseAuthentication yes/g' /etc/ssh/sshd_config
      sudo systemctl reload ssh
    SHELL
  end

end