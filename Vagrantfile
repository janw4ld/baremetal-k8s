# Network prefix to which a single digit is appended so setting
# NETWORK_PREFIX=192.168.1.17 will create the master at 192.168.1.170 
# and workers starting from 192.168.1.171 onward
NETWORK_PREFIX="192.168.1.17"

ENV['VAGRANT_DEFAULT_PROVIDER'] = 'libvirt'
IMAGE_NAME = "roboxes/ubuntu2204"   # 1.5GB, not based on cloud image

WORKER_COUNT = 2    # total nodes = 1 master + WORKER_COUNT

Vagrant.configure("2") do |config|
    config.ssh.insert_key = false
    
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

    config.vm.provision "shell" do |s|
        ssh_prv_key = File.read("/root/.ssh/k8s_ed25519")
        ssh_pub_key = File.readlines("/root/.ssh/k8s_ed25519.pub").first.strip
       
        # https://stackoverflow.com/questions/30075461/how-do-i-add-my-own-public-key-to-vagrant-vm
        s.inline = <<-SHELL
          if ! grep -sq "#{ssh_pub_key}" /home/vagrant/.ssh/authorized_keys; then
            echo "SSH key provisioning."
            mkdir -p /home/vagrant/.ssh/
            touch /home/vagrant/.ssh/authorized_keys
            echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
            echo #{ssh_pub_key} > /home/vagrant/.ssh/id_rsa.pub
            chmod 644 /home/vagrant/.ssh/id_rsa.pub
            echo "#{ssh_prv_key}" > /home/vagrant/.ssh/id_rsa
            chmod 600 /home/vagrant/.ssh/id_rsa
            chown -R vagrant:vagrant /home/vagrant
            exit 0
          fi
        SHELL
      end
end
