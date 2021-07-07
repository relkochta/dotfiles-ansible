# Vagrantfile
# Can be used to try this ansible playbook in a virtual machine;
# see README.md for more details.

Vagrant.configure("2") do |config|
  # Import fedora/34-cloud-base box.
  config.vm.box = "fedora/34-cloud-base"

  # Configure the virtual hardware for the VM.
  config.vm.provider :libvirt do |domain|
    # CPU & RAM
    domain.cpus = 4
    domain.cpu_mode = "host-passthrough"
    domain.memory = "2400"

    # Necessary for communication with guest.
    # You must be in the 'libvirt' group for
    # this to work.
    domain.qemu_use_session = false
    domain.management_network_mode = "none"
  end

  # Ansible Configuration
  config.vm.provision "ansible" do |ansible|
    # The playbook -- points to main.yml.
    ansible.playbook = "main.yml"

    # Set to 'true' if your secrets are encrypted
    # with Ansible Vault.
    ansible.ask_vault_pass = false

    # Uncomment to only run system-wide/user-specific
    # configuration tasks.
    #ansible.tags = "user"
    #ansible.tags = "system"
  end

  # Configure the instance itself.
  config.vm.define :vagrant do |vagrant|
    # Hostname
    # Keep in mind that the playbook will
    # try to reference config-(hostname).yml
    # before using config-default.yml.
    vagrant.vm.hostname = "vagrant"

    # Use a macvtap network interface to access
    # the internet.
    # If a macvtap does not work for some reason
    # (as on some WiFi cards), you can remove this
    # entire section and allow Vagrant to create a
    # network automatically.
    # If you do this, you will need to remove the
    # 'domain.management_network_mode = "none"
    # line above.
    vagrant.vm.network :public_network,
      # Change this to match your main
      # ethernet interface
      :dev => "enp6s0",
      # Don't pollute router's DHCP database
      # with many MAC addresses every time
      # the VM is re-created
      :mac => "aa:bb:cc:dd:ee:ff"
  end
end
