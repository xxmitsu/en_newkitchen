# -*- mode: ruby -*-
# vi: set ft=ruby :
# (c) 2016 - Present Mihai Vultur <xanto@egaming[dot]ro>

Vagrant.require_version '>= 1.8.0'
VAGRANTFILE_API_VERSION = "2"

## HOSTS CONFIGS, Down below..
####### Configs
GLOBAL_CONFIGS = {
  #-- Pin those versions
  software_versions: {
    Chef_DK:       '1.2.20',
    tfenv:         '1.0.1',
    terragrunt:    '0.20.3',
    Packer:        '1.3.2',
    OpenStack_cli: '3.17',
    ShellCheck:    'v0.4.6'
  },
  #-- customize with those
  transfer_local_files: {
    '~/.ssh/id_rsa'                 => '~/.ssh/id_rsa',
    '~/.ssh/id_rsa.pub'             => '~/.ssh/id_rsa.pub',
    'homedir/.bashrc'               => '~/.bashrc',
    'homedir/.git-completion.bash'  => '~/.git-completion.bash',
    'homedir/.git-prompt.bash'      => '~/.git-prompt.bash',
    'homedir/.vimrc'                => '~/.vimrc'
  }
}

####### Vagrant Prerequisites
required_plugins = %w( vagrant-vbguest )
if (/cygwin|mswin|mingw|bccwin|wince|emx/ =~ RUBY_PLATFORM) != nil then
	#-- nfs sharing
	required_plugins.push('vagrant-winnfsd')
end

required_plugins.each do |plugin|
  system "vagrant plugin install #{plugin}" unless Vagrant.has_plugin? plugin
end

def file_dir_or_symlink_exists?(path_to_file)
  File.exist?(File.expand_path(path_to_file)) || File.symlink?(File.expand_path(path_to_file))
end


####### SCRIPTS
install_BASE = <<SCRIPT
  cat /home/vagrant/.ssh/id_rsa.pub >> /home/vagrant/.ssh/authorized_keys
  echo "Installing base ..."
  yum install -y epel-release redhat-lsb ntpdate rpcbind
  true
  #-- we don't need selinux in dev environments
  setenforce 0
  sed -i 's#^SELINUX=.*#SELINUX=permissive#g' /etc/selinux/config

  #Disable IPV6
  echo "net.ipv6.conf.all.disable_ipv6 = 1" >> /etc/sysctl.conf
  echo 1>/proc/sys/net/ipv6/conf/all/disable_ipv6
  echo "search local" > /etc/resolv.conf
  echo "nameserver 8.8.8.8" >> /etc/resolv.conf
  mv /etc/hosts /etc/hosts.old
  echo "192.168.199.20 node1test" > /etc/hosts
  echo "192.168.199.21 node2test" >> /etc/hosts
  echo "127.0.0.1 $(cat /etc/hostname)" >> /etc/hosts
SCRIPT

#--
install_DEV = <<SCRIPT
  echo "Installing DEV ..."
  yum install -y tmux mc vim-enhanced git svn patch unzip gcc ruby rubygems curl bash-completion strace telnet bind-utils tcpdump nc traceroute telnet whois wget pwkickstart python-boto3
SCRIPT

#--
install_chefDK = <<SCRIPT
  echo "Installing ChefDK ..."
  yum install -y https://packages.chef.io/files/stable/chefdk/#{GLOBAL_CONFIGS[:software_versions][:Chef_DK]}/el/7/chefdk-#{GLOBAL_CONFIGS[:software_versions][:Chef_DK]}-1.el7.x86_64.rpm 
  true
SCRIPT

#--
install_DOCKER = <<SCRIPT
  echo "Installing Docker ..."
  wget https://download.docker.com/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
  yum install -y docker-ce
  echo "DOCKER_NETWORK_OPTIONS='-H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock'" | sudo tee /etc/sysconfig/docker-network > /dev/null
  sudo mkdir -p /etc/systemd/system/docker.service.d/
  echo "[Service]" | sudo tee /etc/systemd/system/docker.service.d/service.conf
  echo "EnvironmentFile=-/etc/sysconfig/docker-network" | sudo tee -a /etc/systemd/system/docker.service.d/service.conf
  echo "ExecStart=" | sudo tee -a /etc/systemd/system/docker.service.d/service.conf
  echo "ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock \\$DOCKER_NETWORK_OPTIONS" | sudo tee -a /etc/systemd/system/docker.service.d/service.conf
  sudo systemctl daemon-reload && systemctl enable docker && sudo systemctl restart docker
  echo -e 'export DOCKER_HOST=tcp://127.0.0.1:2375\nunset DOCKER_CERT_PATH\nunset DOCKER_TLS_VERIFY' > /etc/profile.d/docker_TCP.sh
  echo -e 'Run:\n  export DOCKER_HOST=tcp://127.0.0.1:2375 && unset DOCKER_CERT_PATH && unset DOCKER_TLS_VERIFY\nto use docker on the host machine'
SCRIPT

#--
install_TFENV = <<SCRIPT
  echo "Installing TFenv for Terraform ..."
  tfenvDir=/opt/tfenv
  mkdir -p $tfenvDir
  wget https://github.com/vulturm/tfenv/archive/v#{GLOBAL_CONFIGS[:software_versions][:tfenv]}.zip -O /opt/tfenv_#{GLOBAL_CONFIGS[:software_versions][:tfenv]}.zip
  unzip -q -o /opt/tfenv_#{GLOBAL_CONFIGS[:software_versions][:tfenv]}.zip -d ${tfenvDir}
  chown -R vagrant:vagrant ${tfenvDir}
  ln -sf ${tfenvDir}/tfenv-#{GLOBAL_CONFIGS[:software_versions][:tfenv]}/bin/tfenv /usr/bin/tfenv
  ln -sf ${tfenvDir}/tfenv-#{GLOBAL_CONFIGS[:software_versions][:tfenv]}/bin/terraform /usr/bin/terraform
SCRIPT

install_TERRAGRUNT = <<SCRIPT
  echo "Installing Terragrunt ..."
  terragruntDir=/opt/terragrunt
  mkdir -p $terragruntDir
  wget https://github.com/gruntwork-io/terragrunt/releases/download/v#{GLOBAL_CONFIGS[:software_versions][:terragrunt]}/terragrunt_linux_amd64 -O ${terragruntDir}/terragrunt_#{GLOBAL_CONFIGS[:software_versions][:terragrunt]}
  chown -R vagrant:vagrant ${terragruntDir}
  ln -sf ${terragruntDir}/terragrunt_#{GLOBAL_CONFIGS[:software_versions][:terragrunt]} /usr/bin/terragrunt
SCRIPT

#--
install_PACKER = <<SCRIPT
  echo "Installing Packer ..."
  packerDir=/opt/packer
  mkdir -p $packerDir
  wget --content-dis -k https://releases.hashicorp.com/packer/#{GLOBAL_CONFIGS[:software_versions][:Packer]}/packer_#{GLOBAL_CONFIGS[:software_versions][:Packer]}_linux_amd64.zip -O /opt/packer_#{GLOBAL_CONFIGS[:software_versions][:Packer]}_linux_amd64.zip
  unzip -q -o /opt/packer_#{GLOBAL_CONFIGS[:software_versions][:Packer]}_linux_amd64.zip -d ${packerDir}
  chown -R vagrant:vagrant ${packerDir}
  ln -sf ${packerDir}/packer /usr/bin/packer
SCRIPT

#--
install_OPENSTACK = <<SCRIPT
  echo "Installing OpenStack Client ..."
  yum install -y python-devel python-pip
  pip install python-openstackclient==#{GLOBAL_CONFIGS[:software_versions][:OpenStack_cli]}
SCRIPT

#--
install_BASH = <<SCRIPT
  echo "Installing Bash Development Environment ..."
  shellcheckDir=/opt/shellcheck
  mkdir -p $shellcheckDir
  wget -k https://github.com/xxmitsu/dev_bin/blob/master/shellcheck-#{GLOBAL_CONFIGS[:software_versions][:ShellCheck]}.linux.x86_64.tar.xz?raw=true -O ${shellcheckDir}/shellcheck-#{GLOBAL_CONFIGS[:software_versions][:ShellCheck]}.linux.x86_64.tar.xz
  cd ${shellcheckDir} && tar xxf shellcheck-#{GLOBAL_CONFIGS[:software_versions][:ShellCheck]}.linux.x86_64.tar.xz
  chown -R vagrant:vagrant ${shellcheckDir}
  ln -sf ${shellcheckDir}/shellcheck-#{GLOBAL_CONFIGS[:software_versions][:ShellCheck]}/shellcheck /usr/bin/shellcheck
SCRIPT

#--
install_ANSIBLE = <<SCRIPT
  echo "Installing Ansible Development Environment ..."
  yum -y install ansible ansible-doc ansible-lint
SCRIPT

#--
install_ICINGA = <<SCRIPT
  echo "Installing Icinga Test Environment ..."
  wget https://packages.icinga.com/epel/ICINGA-release.repo -O /etc/yum.repos.d/ICINGA-release.repo
  #-- prerequisites
  yum install -y httpd gcc glibc glibc-common gd gd-devel
  #-- dbi
  yum install -y mysql mysql-server libdbi libdbi-devel libdbi-drivers libdbi-dbd-mysql
  service mysqld start
  chkconfig mysqld on

  #-- plugins
  yum install -y nagios-plugins-nrpe nagios-plugins-all nsca perl-snmp

  #-- icinga
  yum -y install icinga-1.14.0-0.el6.x86_64 icinga-gui-config-1.14.0-0.el6.x86_64 icinga-idoutils-libdbi-mysql-1.14.0-0.el6.x86_64 icinga-doc-1.14.0-0.el6.x86_64 icinga-gui-1.14.0-0.el6.x86_64 icinga-idoutils-1.14.0-0.el6.x86_64 libwmf libwmf-lite 

  #-- postconfig
  mkdir -p /dev/shm/icinga/{tmp,checkresults}
  chown -R icinga:icinga /dev/shm/icinga/
  chmod 777 /dev/shm/icinga/{tmp,checkresults}
  echo "CREATE DATABASE IF NOT EXISTS icinga;" | mysql -uroot
  echo "GRANT USAGE ON *.* TO 'icinga'@'localhost' IDENTIFIED BY PASSWORD '*F7EA22C777E1A8D2E1F61A2F9EBBD74FF489FF63';" | mysql -uroot
  echo "GRANT ALL PRIVILEGES ON icinga.* TO 'icinga'@'localhost' WITH GRANT OPTION;" | mysql -uroot
  mysql -uicinga -picinga icinga < /usr/share/doc/icinga-idoutils-libdbi-mysql-1.14.0/db/mysql/mysql.sql

  #-- start
  chkconfig icinga on
  chkconfig httpd on
  chkconfig ido2db on
  service ido2db restart
  service icinga restart
  service httpd restart
SCRIPT

##############################
### VMs
VIRTUAL_MACHINES = {
  workstation: {
    vm_box: 'centos/7',
    hostname: 'workstation.local.lo',
    cpus: 4,
    memory: 2048,
    private_ip: '192.168.199.30',
    environment: 'DevOps',
    shell_script: [ 
      install_BASE,
      install_DEV,
      install_TFENV,
      install_TERRAGRUNT,
      install_PACKER,
      install_ANSIBLE,
      install_OPENSTACK,
      install_BASH,
      install_DOCKER
    ]
  },
  node2test: {
    vm_box: 'centos/7',
    hostname: 'node2test.local.lo',
    cpus: 2,
    memory: 1024,
    private_ip: '192.168.199.21',
    environment: 'DevOps',
    shell_script: [
      install_BASE,
      install_DEV,
      install_ANSIBLE
    ]
  },
  icinga2test: {
    vm_box: 'centos/6',
    hostname: 'icinga2test.local.lo',
    cpus: 1,
    memory: 1024,
    private_ip: '192.168.199.22',
    environment: 'DevOps',
    shell_script: [
      install_BASE,
      install_DEV,
      install_ICINGA
    ]
  },
  centos7test: {
    vm_box: 'centos/7',
    hostname: 'centos7test.local.lo',
    cpus: 2,
    memory: 1000,
    private_ip: '192.168.199.23',
    environment: 'DevOps',
    shell_script: [
      install_BASE,
      install_DEV
    ]
  },

}.freeze

##############################
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  #  config.omnibus.chef_version = '12.8.1'
  config.vbguest.auto_update = false if Vagrant.has_plugin?('vagrant-vbguest')

  VIRTUAL_MACHINES.each do |name, cfg|
    config.vm.box = cfg[:vm_box]
    config.vm.define name do |vm_config|
      vm_config.berkshelf.enabled = false if Vagrant.has_plugin?('vagrant-berkshelf')
      # private net between
      vm_config.vm.network 'private_network', virtualbox__intnet: 'intnet'
      vm_config.vm.network 'private_network', ip: cfg[:private_ip]
      vm_config.vm.hostname = cfg[:hostname]
      vm_config.vm.synced_folder "shared/#{name}", '/vagrant', create: true, type: :nfs
      vm_config.vm.provider 'virtualbox' do |v|
        v.name = cfg[:hostname]
        v.customize ['modifyvm', :id, '--memory', cfg[:memory]]
        v.customize ['modifyvm', :id, '--cpus', cfg[:cpus]]
        v.customize ['modifyvm', :id, '--paravirtprovider', 'default']
        v.customize ['modifyvm', :id, '--ioapic', 'on']
        v.customize ['modifyvm', :id, '--hwvirtex', 'on']
      end
      #--
      if GLOBAL_CONFIGS[:transfer_local_files]
        GLOBAL_CONFIGS[:transfer_local_files].each do |src_name, dst_name|
          vm_config.vm.provision 'file', source: File.expand_path(src_name), destination: dst_name if file_dir_or_symlink_exists?(src_name)
        end
      end
      #--
      if cfg[:shell_script]
	      cfg[:shell_script].each do |provision_with|
		      vm_config.vm.provision :shell, :inline => provision_with
	      end
      end
      #--
      if cfg[:forwarded_ports]
        cfg[:forwarded_ports].each do |port|
          vm_config.vm.network 'forwarded_port', guest: port[:guest], host: port[:host]
        end
      end
    end
  end
end 
