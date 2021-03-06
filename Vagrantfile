# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version.
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

    # use the empty dummy box
    config.vm.box = "dummy"
    config.vm.box_url = "https://github.com/mitchellh/vagrant-aws/raw/master/dummy.box"

    # default provisioning script
    config.vm.provision :shell, path: "./provision/bootstrap.sh"
    
    # language specific Pipelines:
    # Python
    config.vm.provision :shell, path: "./provision/bootstrap_python.sh"
    # PHP
    # config.vm.provision :shell, path: "./provision/bootstrap_php.sh"
    # Java
    # config.vm.provision :shell, path: "./provision/bootstrap_java.sh"
    
    # Use Virtualbox by default
    config.vm.provider "virtualbox"
    config.vm.provider "aws"



    # Configure Virtulbox provider
    config.vm.provider "virtualbox" do |vb, override|

        override.vm.box = "ubuntu/trusty64"
        vb.cpus = 1
        vb.memory = "512"
        vb.gui = false
        
        override.vm.network "forwarded_port", guest: 8080, host: 8080
        override.vm.synced_folder "./data", "/vagrant"
  
        # Configure Vagrant plugin, Cachier
        if Vagrant.has_plugin?("vagrant-cachier")
            # https://github.com/fgrehm/vagrant-cachier
            config.cache.scope = :box
        end
            
        # Configure Vagrant plugin, vbguest auto-upgrade
        if Vagrant.has_plugin?("vagrant-vbguest")
            # https://github.com/dotless-de/vagrant-vbguest
            config.vbguest.auto_update = false
        end
    end



    # configuration Amazon provider
    config.vm.provider :aws do |aws, override|
    
        # Load sensitive AWS credentials from external file, DO NOT save in Repo!!!
        # @see http://blog-osshive.rhcloud.com/2014/02/05/provisioning-aws-instances-with-vagrant/
        require 'yaml'
        aws_filepath = File.dirname(__FILE__) + "/aws-config.yml"
        if File.exist?(aws_filepath)
            aws_config  = YAML.load_file(aws_filepath)["aws"]
        else
            print "Error: '" + aws_filepath + "' is missing...\n"
        end
        
        # set AWS creds
        aws.access_key_id             = aws_config["access_key_id"]
        aws.secret_access_key         = aws_config["secret_access_key"]
        aws.keypair_name              = aws_config["keypair_name"]
        
        # configure SSH... and fuck Windows file pathways...
        override.ssh.private_key_path = aws_config["pemfile"]
        override.ssh.username         = "ubuntu"

        # use Ubuntu / Trust 64-bit HVM
        # @see https://cloud-images.ubuntu.com/locator/ec2/
        aws.region = "us-east-1"
        aws.ami = "ami-fce3c696"
        
        # set instance settings
        # @see https://aws.amazon.com/ec2/instance-types/
        aws.instance_ready_timeout    = 180
        aws.instance_type             = "t2.micro"
        aws.associate_public_ip       = true
        aws.subnet_id                 = aws_config["subnet_id"]
        aws.tags = {
            'Name' => 'jenkins-pipeline-demo',
        }
        # use 40GB, because we like it UUGE!
        aws.block_device_mapping = [
            {
                'DeviceName' => '/dev/xvda',
                'VirtualName' => 'root',
                'Ebs.VolumeSize' => 40,
                'Ebs.DeleteOnTermination' => 'true'
            }
        ]

        # Configure file sharing using rsync.
        # This requires Windows users to have Cygwin or MinGW installed.
        # @see https://www.vagrantup.com/blog/feature-preview-vagrant-1-5-rsync.html
        # @see https://github.com/mitchellh/vagrant/blob/master/website/docs/source/v2/synced-folders/rsync.html.md
        override.vm.synced_folder "./data", "/vagrant", type: "rsync"
        # , disabled: true
        
        # To continuously update files uni-directionally from local host to the
        # remote EC2 instance, open another shell, run:
        # "vagrant rsync-auto"
        
        # Fix for Windows users running Cygwin:
        if Vagrant::Util::Platform.windows?
            ENV["VAGRANT_DETECTED_OS"] = ENV["VAGRANT_DETECTED_OS"].to_s + " cygwin"
        end
    
        # disable the vbguest update plugin
        if Vagrant.has_plugin?("vagrant-vbguest")
            override.vbguest.auto_update = false
        end
    end
end
