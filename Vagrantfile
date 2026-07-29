# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  # Base box for all VMs (Debian 12 (bookworm))
  config.vm.box = "generic/debian12"

  # ─── Jumpbox (administration host) ─────────────────────────────────
  config.vm.define "jumpbox" do |node|
    node.vm.hostname = "jumpbox"
    node.vm.network "private_network", ip: "10.0.0.10"
    node.disksize.size = "10GB"

    node.vm.provider :libvirt do |lv|
      lv.cpus   = 1
      lv.memory = 512
    end
  end

  # ─── Server (Kubernetes control plane) ────────────────────────────
  config.vm.define "server" do |node|
    node.vm.hostname = "server"
    node.vm.network "private_network", ip: "10.240.0.20"
    node.disksize.size = "20GB"

    node.vm.provider :libvirt do |lv|
      lv.cpus   = 1
      lv.memory = 2048
    end
  end

  # ─── Node 0 (Kubernetes worker) ───────────────────────────────────
  config.vm.define "node-0" do |node|
    node.vm.hostname = "node-0"
    node.vm.network "private_network", ip: "10.240.0.30"
    node.disksize.size = "20GB"

    node.vm.provider :libvirt do |lv|
      lv.cpus   = 1
      lv.memory = 2048
    end
  end

  # ─── Node 1 (Kubernetes worker) ───────────────────────────────────
  config.vm.define "node-1" do |node|
    node.vm.hostname = "node-1"
    node.vm.network "private_network", ip: "10.240.0.31"
    node.disksize.size = "20GB"

    node.vm.provider :libvirt do |lv|
      lv.cpus   = 1
      lv.memory = 2048
    end
  end
end
