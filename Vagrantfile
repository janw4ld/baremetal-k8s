# Network prefix to which a single digit is appended so setting
# NETWORK_PREFIX=192.168.1.17 will create the master at 192.168.1.170 
# and workers starting from 192.168.1.171 onward
NETWORK_PREFIX="192.168.1.17"

ENV['VAGRANT_DEFAULT_PROVIDER'] = 'libvirt'
IMAGE_NAME = "roboxes/ubuntu2204"   # 1.5GB, not based on cloud image

WORKER_COUNT = 2    # total nodes = 1 master + WORKER_COUNT

Vagrant.configure("2") do |config|
    config.vm.provider :libvirt do |libvirt|
        libvirt.cpus = 2
        libvirt.memory = 4096
        libvirt.driver = "kvm"
    end

    config.vm.define "master" do |master|
        master.vm.box = IMAGE_NAME
	    master.vm.network :public_network,
            :dev => "br0",  # check github.com/janw4ld/baremetal-k8s#network-bridge
            :mode => "bridge",
            :type => "bridge",
	        :ip => "#{NETWORK_PREFIX}0"
        master.vm.hostname = "master"
    end

    (1..WORKER_COUNT).each do |i|
        config.vm.define "worker-#{i}" do |worker|
            worker.vm.box = IMAGE_NAME
            worker.vm.network :public_network,
                :dev => "br0",
                :mode => "bridge",
                :type => "bridge",
                :ip => "#{NETWORK_PREFIX}#{i}"
            worker.vm.hostname = "worker-#{i}"
        end
    end
end
