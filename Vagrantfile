Vagrant.configure("2") do |config|

	config.vm.box = "ubuntu/trusty64"
	config.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"
	config.vm.provider "virtualbox" do |vb|
		vb.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/vagrant", "1"]
        vb.customize ["guestproperty", "set", :id, "/VirtualBox/GuestAdd/VBoxService/--timesync-set-threshold", 10000]
	end

	config.vm.provision "shell", inline: <<-END
		apt-add-repository ppa:ansible/ansible
		apt-get -y autoremove
		apt-get update

		#
		# Install NTP, Git, unzip, Python, Ansible, Ruby
		#
		apt-get install -y ntp git unzip python software-properties-common ansible ruby-full
		apt-get install -y python python-pip python-dev build-essential
        pip install --upgrade pip
        pip install --upgrade virtualenv
		pip install docker-py
		apt-get -y autoremove

		#
		# Install Terraform
		#
		wget -q https://releases.hashicorp.com/terraform/0.6.14/terraform_0.6.14_linux_amd64.zip
		mkdir -p /opt/terraform
		unzip -oq terraform_0.6.14_linux_amd64.zip -d /opt/terraform
		rm -f terraform_0.6.14_linux_amd64.zip
		echo "export PATH=$PATH:/opt/terraform" >> /etc/bash.bashrc
		export PATH=$PATH:/opt/terraform

		cd /vagrant
		terraform get
		terraform apply

		echo Sleeping for 120 seconds while the wiremock initializes.
		sleep 120
		echo Performing health check
		echo Connecting to http://`terraform output -module=wiremock ip`:80/api/healthCheck
		curl http://`terraform output -module=wiremock ip`:80/api/healthCheck
	END

end
