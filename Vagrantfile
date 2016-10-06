# -*- mode: ruby -*-
# vi: set ft=ruby :

$script = <<SCRIPT

export DEBIAN_FRONTEND=noninteractive

# 1. Update server
echo "deb http://us.archive.ubuntu.com/ubuntu/ trusty multiverse" | sudo tee -a /etc/apt/sources.list
apt-get -y update
apt-get -y dist-upgrade

# Ensure UTF-8
apt-get -y install language-pack-en
update-locale LANG="en_US.UTF-8"
locale-gen en_US.UTF-8
LANG       en_US.UTF-8
LC_ALL     en_US.UTF-8

# 2. Install PPA for LibreOffice 4.4 and libsslAnchor link for: install ppa for libreoffice 44 and libssl
apt-get -y install software-properties-common
add-apt-repository ppa:libreoffice/libreoffice-4-4
add-apt-repository -y ppa:ondrej/php

# 3. Install key for BigBlueButton
apt-get -y install wget
wget http://ubuntu.bigbluebutton.org/bigbluebutton.asc -O- | sudo apt-key add -
echo "deb http://ubuntu.bigbluebutton.org/trusty-1-0/ bigbluebutton-trusty main" | sudo tee /etc/apt/sources.list.d/bigbluebutton.list
apt-get -y update

# 4. Install ffmpeg
apt-get -y install build-essential git-core checkinstall yasm texi2html libvorbis-dev libx11-dev libvpx-dev libxfixes-dev zlib1g-dev pkg-config netcat libncurses5-dev
FFMPEG_VERSION=2.3.3
cd /usr/local/src
if [ ! -d "/usr/local/src/ffmpeg-${FFMPEG_VERSION}" ]; then
  wget "http://ffmpeg.org/releases/ffmpeg-${FFMPEG_VERSION}.tar.bz2"
  tar -xjf "ffmpeg-${FFMPEG_VERSION}.tar.bz2"
fi
cd "ffmpeg-${FFMPEG_VERSION}"
./configure --enable-version3 --enable-postproc --enable-libvorbis --enable-libvpx
make
checkinstall --pkgname=ffmpeg --pkgversion="5:${FFMPEG_VERSION}" --backup=no --deldoc=yes --default

# 5. Install BigBlueButton
apt-get -y install bigbluebutton
apt-get -y install bbb-demo
bbb-conf --enablewebrtc

apt-get -y -q autoremove
apt-get -y clean
SCRIPT

$startup_script = <<SCRIPT

export DEBIAN_FRONTEND=noninteractive

bbb-conf --clean
bbb-conf --check
bbb-conf --setip $(ifconfig eth1 | grep 'inet addr:' | cut -d: -f2 | awk '{ print $1}')

echo "Finish setup on http://$(ifconfig eth1 | grep 'inet addr:' | cut -d: -f2 | awk '{ print $1}')"
SCRIPT


# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "ubuntu/trusty64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  config.vm.network "forwarded_port", guest: 80, host: 80
  config.vm.network "forwarded_port", guest: 9123, host: 9123
  config.vm.network "forwarded_port", guest: 1935, host: 1935
  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
    # Display the VirtualBox GUI when booting the machine
    vb.gui = false
  
    # Customize the amount of memory on the VM:
    vb.memory = "4096"
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL

  config.vm.provision :shell, inline: $script

  config.vm.provision "fix-no-tty", type: "shell" do |s|
      s.privileged = false
      s.inline = "sudo sed -i '/tty/!s/mesg n/tty -s \\&\\& mesg n/' /root/.profile"
  end

  config.vm.provision :shell, inline: $startup_script, run: "always"
end
