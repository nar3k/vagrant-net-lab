# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

## Generate a unique ID for this project
UUID = "OTYUID"

## Define ports mapping to create a Full Mesh between all 4 vqfx
ports_map = { 'leaf01' => [1,2,3,4,13],
              'leaf02' => [5,6,7,8,9,10,14],
              'spine01' => [1,2,5,6],
              'spine02' => [3,4,7,8],
              'exit01' => [9,10,11,15],
              'r01'=>[11,12,16],
               }
host_port_map = { 'dc-host01' => 13,
                   'dc-host02' => 14,
                   'dmz-host01' => 15,
                   'ext-host01' => 16,

}
vqfx_devices = ['leaf01','leaf02','spine01','spine02','exit01']
vsrx_devices = ['r01']
host_devices = ['dc-host01','dc-host02', 'dmz-host01','ext-host01']



Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

    config.ssh.insert_key = false

    vqfx_devices.each do |id|
        re_name  = ( id ).to_sym
        pfe_name = ( id + "-pfe" ).to_sym

        # ##############################
        # ## Packet Forwarding Engine ##
        # ##############################
        config.vm.define pfe_name do |vqfxpfe|
            vqfxpfe.ssh.insert_key = false
            vqfxpfe.vm.box = 'juniper/vqfx10k-pfe'

            # DO NOT REMOVE / NO VMtools installed
            vqfxpfe.vm.synced_folder '.', '/vagrant', disabled: true
            vqfxpfe.vm.network 'private_network', auto_config: false, nic_type: '82540EM', virtualbox__intnet: "#{UUID}_vqfx_internal_#{id}"

            # In case you have limited resources, you can limit the CPU used per vqfx-pfe VM, usually 50% is good
            # vqfxpfe.vm.provider "virtualbox" do |v|
            #    v.customize ["modifyvm", :id, "--cpuexecutioncap", "50"]
            # end
        end

        ##########################
        ## Routing Engine  #######
        ##########################
        config.vm.define re_name do |vqfx|
            vqfx.vm.hostname = "#{id}"
            vqfx.vm.box = 'juniper/vqfx10k-re'

            # DO NOT REMOVE / NO VMtools installed
            vqfx.vm.synced_folder '.', '/vagrant', disabled: true

            # Management port
            vqfx.vm.network 'private_network', auto_config: false, nic_type: '82540EM', virtualbox__intnet: "#{UUID}_vqfx_internal_#{id}"
            vqfx.vm.network 'private_network', auto_config: false, nic_type: '82540EM', virtualbox__intnet: "#{UUID}_reserved_bridge"

            # Dataplane ports
            ports_map[id].each do |seg_id|
               vqfx.vm.network 'private_network', auto_config: false, nic_type: '82540EM', virtualbox__intnet: "#{UUID}_seg#{[seg_id]}"
            end
        end
    end
    ##VQFX
    vsrx_devices.each do |id|
         re_name = ( id ).to_sym
         config.vm.define re_name do |vsrx|
             vsrx.vm.hostname = "#{id}"
             vsrx.vm.box = "juniper/ffp-12.1X47-D15.4"
             ports_map[id].each do |seg_id|
                vsrx.vm.network 'private_network', auto_config: false, nic_type: '82540EM', virtualbox__intnet: "#{UUID}_seg#{[seg_id]}"
            end
        end
    end

    host_devices.each do |id|
    ##SERVERS
    srv_name = ( id ).to_sym
     config.vm.define srv_name do |srv|
       srv.vm.box = "robwc/minitrusty64"
       srv.vm.hostname = "#{id}"
       srv.vm.network 'private_network', ip: "172.16.#{host_port_map[id]}.2", nic_type: '82540EM', virtualbox__intnet: "#{UUID}_server_#{host_port_map[id]}"
       srv.ssh.insert_key = true
       srv.vm.provision "shell",
           inline: "sudo route add -net 172.16.0.0 netmask 255.255.0.0 gw 172.16.#{host_port_map[id]}}.1"
     end
end
    ##############################
    ## Box provisioning        ###
    ## exclude Windows host    ###
    ##############################
    if !Vagrant::Util::Platform.windows?
        config.vm.provision "ansible" do |ansible|
            ansible.groups = {
                "vqfx10k-pfe"  => ["leaf01-pfe", "leaf01-pfe","spine01-pfe","spine02-pfe","exit01-pfe"],
                "hosts" => ['dc-host01','dc-host02', 'dmz-host01','ext-host01'],
                "leaf" => ["leaf01","leaf02"],
                "spine" => ["spine01","spine02"],
                "edge" => ["exit01","r01"],
                "all:children" => ["leaf", "spine","vqfx10k-pfe","hosts","spine"]
            }
            ansible.playbook = "pb.conf.all.commit.yaml"
        end
    end
end
