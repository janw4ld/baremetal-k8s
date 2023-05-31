# Network prefix to which a single digit is appended so setting
# NETWORK_PREFIX=192.168.1.17 will create the master at 192.168.1.170 
# and workers starting from 192.168.1.171 onward
NETWORK_PREFIX="192.168.1.17"

ENV['VAGRANT_DEFAULT_PROVIDER'] = 'libvirt'
IMAGE_NAME = "roboxes/ubuntu2204"   # 1.5GB, not based on cloud image

WORKER_COUNT = 2    # total nodes = 1 master + WORKER_COUNT

HOST_HOME_PATH = "/root"

Vagrant.configure("2") do |config|
    config.ssh.insert_key = false
    ssh_key_path = "#{HOST_HOME_PATH}/.ssh/k8s_ed25519"
    ssh_prv_key = File.read(ssh_key_path)
    ssh_pub_key = File.readlines("#{ssh_key_path}.pub").first.strip

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

    system(
    <<~TEXT
        echo '[kube-masters]
        master.kube.local ansible_host=#{NETWORK_PREFIX}0
        
        [kube-workers]' >./hosts
    TEXT
    )

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

        system(
            "echo 'worker-#{i}.kube.local ansible_host=#{NETWORK_PREFIX}#{i}' \
            >>./hosts"
        )
    end

    system(
    <<~TEXT
        echo '[kube-masters:vars]
        ansible_ssh_port=22
        ansible_ssh_user=vagrant
        ansible_ssh_private_key_file=#{ssh_key_path}

        [kube-workers:vars]
        ansible_ssh_port=22
        ansible_ssh_user=vagrant
        ansible_ssh_private_key_file=#{ssh_key_path}

        [ubuntu:children]
        kube-masters
        kube-workers' >>./hosts
    TEXT
    )

    config.vm.provision "shell" do |s|       
        # https://stackoverflow.com/questions/30075461/how-do-i-add-my-own-public-key-to-vagrant-vm
        s.inline = <<-SHELL
          if ! grep -sq "#{ssh_pub_key}" /home/vagrant/.ssh/authorized_keys; then
            echo "SSH key provisioning."
            mkdir -p /home/vagrant/.ssh/
            touch /home/vagrant/.ssh/authorized_keys
            echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
            chown -R vagrant:vagrant /home/vagrant/.ssh/
          fi
        SHELL
    end

    (0..WORKER_COUNT).each do |i|   # add vms to known_hosts to avoid prompt
        system(
            "ssh-keyscan -H #{NETWORK_PREFIX}#{i} 2>/dev/null \
            | tee -a #{HOST_HOME_PATH}/.ssh/known_hosts >/dev/null"
        )
    end
end
