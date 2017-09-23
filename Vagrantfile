# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

## Generate a unique ID for this project
UUID = "OTYUID"

## Define ports mapping to create a Full Mesh between all 4 vqfx
ports_map = { 'leaf01' => [1,2,3,4,27],
              'leaf02' => [5,6,7,8,9,10],
              'spine01' => [1,2,5,6],
              'spine02' => [3,4,7,8],
              'exit01' => [9,10],
               }
junos_devices = ['leaf01','leaf02','spine01','spine02','exit01']

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

    config.ssh.insert_key = false

    junos_devices.each do |id|
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

    ##############################
    ## Box provisioning        ###
    ## exclude Windows host    ###
    ##############################
    if !Vagrant::Util::Platform.windows?
        config.vm.provision "ansible" do |ansible|
            ansible.groups = {
                "vqfx10k" => ["vqfx1", "vqfx2" ],
                "vqfx10k-pfe"  => ["vqfx1-pfe", "vqfx2-pfe"],
                "all:children" => ["vqfx10k", "vqfx10k-pfe"]
            }
            ansible.playbook = "pb.conf.all.commit.yaml"
        end
    end
end
