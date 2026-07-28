# -*- mode: ruby -*-
require 'fileutils'

BOX = "generic/debian12"

MACHINES = {
  "jumpbox" => { cpus: 1, memory: 512,  disk: 10 },
  "server"  => { cpus: 1, memory: 2048, disk: 20 },
  "node-0"  => { cpus: 1, memory: 2048, disk: 20 },
  "node-1"  => { cpus: 1, memory: 2048, disk: 20 },
}

IPS = {
  "jumpbox" => "192.168.56.10",
  "server"  => "192.168.56.11",
  "node-0"  => "192.168.56.12",
  "node-1"  => "192.168.56.13",
}

# Build /etc/hosts block for cluster machines (excludes jumpbox,
# matches the guide's "hosts" file content).
HOSTS_ENTRIES = IPS.reject { |k,_| k == "jumpbox" }
                   .map { |name, ip| "#{ip} #{name}.kubernetes.local #{name}" }
                   .join("\n")

KEY_DIR = File.join(File.dirname(__FILE__), ".kthw-keys")
PRIVATE_KEY = File.join(KEY_DIR, "id_ed25519")
PUBLIC_KEY  = File.join(KEY_DIR, "id_ed25519.pub")

FileUtils.mkdir_p(KEY_DIR)
unless File.exist?(PRIVATE_KEY)
  system("ssh-keygen -t ed25519 -f #{PRIVATE_KEY} -N '' -C 'kthw-lab' > /dev/null")
end

Vagrant.configure("2") do |config|
  MACHINES.each do |name, specs|
    config.vm.define name do |machine|
      machine.vm.box = BOX
      machine.vm.hostname = name

      machine.vm.provider :libvirt do |lv|
        lv.cpus = specs[:cpus]
        lv.memory = specs[:memory]
        lv.machine_virtual_size = specs[:disk]
        lv.default_prefix = ""
      end

      machine.vm.network :private_network,
        ip: IPS[name],
        netmask: "255.255.255.0",
        :libvirt__network_name => "kthw"

      machine.vm.synced_folder ".", "/vagrant", disabled: true

      # ── public key → all machines ──────────────────────────────
      machine.vm.provision "file",
        source: PUBLIC_KEY,
        destination: "/tmp/kthw_id_ed25519.pub"

      machine.vm.provision "shell", inline: <<-SHELL
        mkdir -p /home/vagrant/.ssh
        cat /tmp/kthw_id_ed25519.pub >> /home/vagrant/.ssh/authorized_keys
        sort -u /home/vagrant/.ssh/authorized_keys -o /home/vagrant/.ssh/authorized_keys
        chmod 700 /home/vagrant/.ssh
        chmod 600 /home/vagrant/.ssh/authorized_keys
        chown -R vagrant:vagrant /home/vagrant/.ssh
      SHELL

      # ── root SSH access (server, node-0, node-1) ───────────────
      if name != "jumpbox"
        machine.vm.provision "shell", inline: <<-SHELL
          sed -i 's/^#*PermitRootLogin.*/PermitRootLogin yes/' /etc/ssh/sshd_config
          systemctl restart sshd

          mkdir -p /root/.ssh
          cat /tmp/kthw_id_ed25519.pub >> /root/.ssh/authorized_keys
          sort -u /root/.ssh/authorized_keys -o /root/.ssh/authorized_keys
          chmod 700 /root/.ssh
          chmod 600 /root/.ssh/authorized_keys
        SHELL
      end

      # ── root SSH client config (jumpbox) ───────────────────────
      if name == "jumpbox"
        machine.vm.provision "file",
          source: PRIVATE_KEY,
          destination: "/home/vagrant/.ssh/id_ed25519"

        machine.vm.provision "shell", inline: <<-SHELL
          chmod 600 /home/vagrant/.ssh/id_ed25519
          chown vagrant:vagrant /home/vagrant/.ssh/id_ed25519

          cat > /home/vagrant/.ssh/config <<-'SSHCONF'
Host server node-0 node-1
  StrictHostKeyChecking no
  UserKnownHostsFile /dev/null
SSHCONF
          chown vagrant:vagrant /home/vagrant/.ssh/config
          chmod 600 /home/vagrant/.ssh/config

          mkdir -p /root/.ssh
          cp /home/vagrant/.ssh/id_ed25519 /root/.ssh/id_ed25519
          cp /home/vagrant/.ssh/config /root/.ssh/config
          chmod 600 /root/.ssh/id_ed25519 /root/.ssh/config
        SHELL
      end

      # ── FQDN via 127.0.1.1 (all machines) ──────────────────────
      machine.vm.provision "shell", inline: <<-SHELL
        sed -i "s/^127.0.1.1.*/127.0.1.1\\t#{name}.kubernetes.local #{name}/" /etc/hosts
      SHELL

      # ── /etc/hosts entries for cluster machines ────────────────
      machine.vm.provision "shell", inline: <<-SHELL
        if ! grep -q "#{IPS["server"]}" /etc/hosts; then
          cat >> /etc/hosts <<-'HOSTS'

# Kubernetes The Hard Way
#{HOSTS_ENTRIES}
HOSTS
        fi
      SHELL
    end
  end
end
