Vagrant.configure("2") do |config|
  config.vm.box = "boxomatic/centos-stream-9"
  config.vm.network "private_network", ip: "192.168.33.11"
  # 设置虚拟机的主机名
  config.vm.hostname="hive"
  config.vm.provider "virtualbox" do |vb|
    vb.name = "hive"
    vb.gui = true
    vb.memory = "2048"
  end
end