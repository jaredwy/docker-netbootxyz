# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  # --- CONFIGURATION TOGGLE ---
  use_local_files = ENV["NETBOOT_LOCAL_FILES"] == "true"

  common_setup = lambda do |machine|
    machine.vm.box = "bento/ubuntu-24.04"
    machine.vm.network "public_network"
    # Was having issues with using the default sync on virtualbox on osx
    if use_local_files
      machine.vm.synced_folder ".", "/vagrant",
        type: "rsync",
        rsync__exclude: [".git/", ".github/", ".vagrant" "docs"],
        rsync__args: ["--verbose", "--archive", "--delete", "-z"]
    else
      machine.vm.synced_folder ".", "/vagrant", disabled: true
    end
  end

  # --- DEMO MACHINE (The PXE Client) ---
  config.vm.define :demo, primary: true do |demo|
    common_setup.call(demo)
    demo.vm.boot_timeout = 5
    demo.vm.provider "virtualbox" do |vb|
        vb.gui = true
        vb.memory = 5120
        vb.customize ["modifyvm", :id, "--boot1", "net"]
        vb.customize ["modifyvm", :id, "--nic1", "none"]
    end
  end

  # --- NETBOOT MACHINE (The Docker Host) ---
  config.vm.define :netboot, autostart: false do |netboot|
    common_setup.call(netboot)

    dhcp_range_start = ENV.fetch("DHCP_RANGE_START", "192.168.0.1")

    netboot.vm.provision "docker" do |d|
      if use_local_files
        d.build_image "/vagrant", args: "-t local-netboot"
        image_tag = "local-netboot"
      else
        image_tag = "samdbmg/dhcp-netboot.xyz"
      end

      d.run image_tag,
        args: "--net=host --cap-add=NET_ADMIN -e DHCP_RANGE_START=#{dhcp_range_start}"
    end
  end

  # Global Proxy Settings
  if Vagrant.has_plugin?("vagrant-proxyconf")
    config.proxy.http     = ENV["http_proxy"]
    config.proxy.https    = ENV["https_proxy"]
    config.proxy.no_proxy = ENV["no_proxy"]
  end
end
