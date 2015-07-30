# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'
pconfig = YAML::load_file(File.join(File.dirname(File.expand_path(__FILE__)), '/group_vars/all/project.yml'))
dconfig = YAML::load_file(File.join(File.dirname(File.expand_path(__FILE__)), '/group_vars/all/developer.yml'))

Vagrant.configure("2") do |config|
  
  config.vm.provider "virtualbox" do |v|
    v.name = pconfig["PROJECT_OWNER"]
    v.customize ["modifyvm", :id, "--memory", dconfig["MEMORY"]]
    v.customize ["modifyvm", :id, "--cpuexecutioncap", dconfig["CPUEXECUTIONCAP"]]
    # This is only necessary if your CPU does not support VT-x or you run virtualbox
    # inside virtualbox
    v.customize ["modifyvm", :id, "--vtxvpid", dconfig["VTX"]]
    v.customize ["modifyvm", :id, "--natdnshostresolver1", "off"]
    v.customize ["modifyvm", :id, "--natdnsproxy1", "off"]

  end
    config.vm.hostname = pconfig["PROJECT_OWNER"]+".dev"

    config.vm.box = dconfig["BOX"]
    config.vm.box_url = dconfig["BOX_URL"]

    # Synced Folders
    # Windows
    if RbConfig::CONFIG['host_os'] =~ /cygwin|mswin|mingw/
      if File.directory?('../src')
          config.vm.synced_folder "..\\..\\build", "/home/"+pconfig["VM_HOME"]+"", :nfs => true
      else
          config.vm.synced_folder "..\\web", "/home/"+pconfig["VM_HOME"]+"", :nfs => true
      end  
    # Mac/Linux    
    else
        config.vm.synced_folder "../web", "/home/"+pconfig["VM_HOME"]+"", :nfs => true
    end

    if dconfig["PLUGINS"]["TRIGGERS"] = true         
      config.trigger.after [:up, :resume], :stdout => true do
        info "Executing bundle install action on the VM..."
        run  "vagrant ssh -c 'cd /home/kc/kahunacoder.com/site && bundle install'"
      end
      config.trigger.after [:up, :resume], :stdout => true do
        info "Executing bundle install action on the VM..."
        run  "vagrant ssh -c 'cd /home/kc/kahunacoder.com/site && bundle update'"
      end
    end
          

  # Defaults for typical installations
    config.ssh.forward_agent = true
    config.vm.network :forwarded_port, guest: dconfig["BROWSER_SYNC_PORT"], host: dconfig["BROWSER_SYNC_PORT"], auto_correct: true
    config.vm.network :forwarded_port, guest: dconfig["BROWSER_SYNC_UI_PORT"], host: dconfig["BROWSER_SYNC_UI_PORT"], auto_correct: true
    config.vm.network "forwarded_port", guest: 4000, host: 4000
  # Plugin settings
    
    config.vbguest.auto_update = dconfig["PLUGINS"]["VBGUEST"]
    config.box_updater.autoupdate = dconfig["PLUGINS"]["BOX_UPDATER"]
 
    if dconfig["PLUGINS"]["HOSTSUPDATER"] = true
      config.hostsupdater.remove_on_suspend = false
    end
    if dconfig["PLUGINS"]["EXEC"] = true
      config.exec.commands '*', directory: '/home/'+pconfig["VM_HOME"]+'/site'
    end
    if dconfig["PLUGINS"]["AUTO_NETWORK"] = true
      config.vm.network  :private_network, :ip =>'0.0.0.0', :auto_network => true
    end
  	
  config.vm.provision :ansible do |ansible|
    # ansible.verbose = "vvvv"
    ansible.playbook = "playbook.yml"
  end
end
