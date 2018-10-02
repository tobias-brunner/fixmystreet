# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'getoptlong'
opts = GetoptLong.new(
  [ '--base-box', GetoptLong::OPTIONAL_ARGUMENT ]
)

baseBox='mysociety/fixmystreet'
opts.each do |opt, arg|
  case opt
    when '--base-box'
      baseBox=arg
  end
end

$setup = <<-EOS
    BASEBOX=$1
    # To prevent "dpkg-preconfigure: unable to re-open stdin: No such file or directory" warnings
    export DEBIAN_FRONTEND=noninteractive
    # Make sure git submodules are checked out!
    echo "Checking submodules exist/up to date"
    if [ ! "$BASEBOX" = "mysociety/fixmystreet" ]; then
        # Don't assume git is installed.
        apt-get -qq install -y git >/dev/null
    fi
    cd fixmystreet
    git submodule --quiet update --init --recursive --rebase
    if [ "$BASEBOX" = "mysociety/fixmystreet" ]; then
        [ ! -e /home/vagrant/fixmystreet/local ] && mkdir /home/vagrant/fixmystreet/local
        mount -o bind /usr/share/fixmystreet/local /home/vagrant/fixmystreet/local
        chown -R vagrant:vagrant /home/vagrant/fixmystreet/local
    fi
    cd commonlib
    git config core.worktree "../../../commonlib"
    echo "gitdir: ../.git/modules/commonlib" > .git
    cd ../..
    # Fetch and run install script
    wget -O install-site.sh --no-verbose https://github.com/mysociety/commonlib/raw/master/bin/install-site.sh
    sh install-site.sh --dev fixmystreet vagrant 127.0.0.1.xip.io
    SUCCESS=$?
    # Even if it failed somehow, we might as well update the port if possible
    if [ -e fixmystreet/conf/general.yml ]; then
        # We want to be on port 3000 for development
        sed -i -r -e "s,^( *BASE_URL: .*)',\\1:3000'," fixmystreet/conf/general.yml
    fi
    # Create a superuser for the admin
    su vagrant -c 'fixmystreet/bin/createsuperuser superuser@example.org password'
    if [ $SUCCESS -eq 0 ]; then
        # All done
        echo "****************"
        echo "You can now ssh into your vagrant box: vagrant ssh"
        echo "The website code is found in: ~/fixmystreet"
        echo "You can run the dev server with: script/server"
        echo "Access the admin with username: superuser@example.org and password: password"
    else
        echo "Unfortunately, something appears to have gone wrong with the installation."
        echo "Please see above for any errors, and do ask on our mailing list for help."
        exit 1
    fi
EOS

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # All Vagrant configuration is done here. The most common configuration
  # options are documented and commented below. For a complete reference,
  # please see the online documentation at vagrantup.com.

  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.box = "#{baseBox}"

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  config.vm.network :forwarded_port, guest: 3000, host: 3000
  # And 3001 for the Cypress test server
  config.vm.network :forwarded_port, guest: 3001, host: 3001

  config.vm.synced_folder ".", "/home/vagrant/fixmystreet", :owner => "vagrant", :group => "vagrant"

  config.vm.provision "shell" do |s|
    s.inline = $setup
    s.args   = "#{baseBox}"
  end

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network :private_network, ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network :public_network

  # If true, then any SSH connections made will enable agent forwarding.
  # Default value: false
  # config.ssh.forward_agent = true

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider :virtualbox do |vb|
  #   # Don't boot with headless mode
  #   vb.gui = true
  #
  #   # Use VBoxManage to customize the VM. For example to change memory:
  #   vb.customize ["modifyvm", :id, "--memory", "1024"]
  # end
  #
  # View the documentation for the provider you're using for more
  # information on available options.

end
