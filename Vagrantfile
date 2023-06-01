# The network prefix to which a single digit is appended, i.e. setting
# NETWORK_PREFIX=192.168.1.17 will create a master node at 192.168.1.170 and
# workers starting from 192.168.1.171 onward
NETWORK_PREFIX="192.168.1.17"

ENV['VAGRANT_DEFAULT_PROVIDER'] = 'libvirt'
IMAGE_NAME = "roboxes/ubuntu2204"   # 1.5GB, not based on cloud image

WORKER_COUNT = 2    # total nodes = 1 master + WORKER_COUNT

HOST_HOME_PATH = "/home/meka"
SSH_KEY_PATH = "#{HOST_HOME_PATH}/.ssh/k8s_ed25519"


Vagrant.configure("2") do |config|
    config.ssh.insert_key = false

    ssh_prv_key = File.read(SSH_KEY_PATH)
    ssh_pub_key = File.readlines("#{SSH_KEY_PATH}.pub").first.strip

    config.vm.provider :libvirt do |libvirt|
        libvirt.cpus = 2
        libvirt.memory = 4096
        libvirt.driver = "kvm"
    end

    node_names = ["master"] + (1..WORKER_COUNT).map { |i| "worker-#{i}" }

    node_names.each_with_index do |node_name,index|
        config.vm.define node_name do |node|
            node.vm.box = IMAGE_NAME
            node.vm.network :public_network,
            :dev => "br0",  # check github.com/janw4ld/baremetal-k8s#network-bridge
            :mode => "bridge",
            :type => "bridge",
            :ip => "#{NETWORK_PREFIX}#{index}"
            node.vm.hostname = node_name
        end
    end

    File.write('./hosts',
    <<~TEXT
        [kube-masters]
        master.kube.local ansible_host=#{NETWORK_PREFIX}0
        
        [kube-workers]
        #{node_names.drop(1).map.with_index { |node_name, index| index += 1
            "worker-#{index}.kube.local ansible_host=#{NETWORK_PREFIX}#{index}"
        }.join("\n")}

        [kube-masters:vars]
        ansible_ssh_port=22
        ansible_ssh_user=vagrant
        ansible_ssh_private_key_file=#{SSH_KEY_PATH}

        [kube-workers:vars]
        ansible_ssh_port=22
        ansible_ssh_user=vagrant
        ansible_ssh_private_key_file=#{SSH_KEY_PATH}

        [ubuntu:children]
        kube-masters
        kube-workers
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
