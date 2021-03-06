!SLIDE subsection center transition=scrollVert
#Desarrollando con Chef

!SLIDE smbullets transition=scrollVert
# Desarrollando con Chef
* Se respeta una filosofía muy instaurada en la comunidad Ruby
	* **Convention over configuration**
* Todos los cookbooks tienen la misma estructura

!SLIDE commandline transition=scrollVert
# Estructura de un cookbook

	$ tree -d mis_cookbooks/mi_cookbook
	 ├── attributes
	 ├── definitions
	 ├── files
	 ├── libraries
	 ├── providers
	 ├── recipes
	 ├── resources
	 └── templates
	
!SLIDE smbullets small transition=scrollVert
# Estructura de un cookbook
* Los **recipes** se guardan en el directorio `recipes`. Y su nombre es importante:
  * **recipe[ntp::foo]:** `mis_cookbooks/ntp/recipes/foo.rb`
  * **recipe[ntp]:** `mis_cookbooks/ntp/recipes/default.rb`
* El nombre de un recipe debe especificar el nombre del cookbook definendo así un **namespace**

!SLIDE commandline incremental transition=scrollVert
# Mi primer recipe

Creamos el directorio para el `cookbook` y `recipe`
 
	$ mkdir -p prueba_chef/cookbooks/test/recipes

!SLIDE small transition=scrollVert
# Mi primer recipe

Creamos el archivo `prueba_chef/cookbooks/test/recipes/default.rb` con el siguiente contenido:

	@@@ ruby
	user "ejemplo" do
	  home '/home/ejemplo'
	  shell '/bin/bash'
	  password 'STHx70ea82L.Yw1eAha15.'
	  # calculada con: 
	  #   openssl passwd -1 "pass_de_ejemplo"
	end

!SLIDE commandline incremental smaller transition=scrollVert
# Lo probamos usando Vagrant

Ubicados dentro del directorio raiz `prueba_chef/` corremos

	$ vagrant init ubuntu-1204 \
		 http://vagrantbox.desarrollo.cespi.unlp.edu.ar/pub/ubuntu-12.04.2-cespi-amd64.box
	A `Vagrantfile` has been placed in this directory. You are now
	ready to `vagrant up` your first virtual environment! Please read
	the comments in the Vagrantfile as well as documentation on
	`vagrantup.com` for more information on using Vagrant.

!SLIDE smaller transition=scrollVert
# Lo probamos usando Vagrant

## Editamos el archivo generado
El comando vagrant init creó un archivo a partir de un template que llamó `Vagrantfile`. Debemos indicar qué IP darle a nuestra máquina virtual. Utilizaremos `33.33.33.100`

	@@@ ruby
	Vagrant.configure("2") do |config|
		config.vm.box = "opscode-ubuntu-1204"
		config.vm.box_url = "http://vagra..."
		config.vm.network :private_network, ip: "33.33.33.100"
	end

!SLIDE commandline incremental transition=scrollVert
# Lo probamos usando Vagrant

Ahora podemos iniciar la máquina con vagrant
	
	$ vagrant up
	Bringing machine 'default' up with 'virtualbox' provider...
	[default] Box 'opscode-ubuntu-1204' was not found. Fetching box from 
	specified URL for the provider 'virtualbox'. Note that if the URL does not 
	have a box for this provider, you should interrupt Vagrant now and add
	the box yourself. Otherwise Vagrant will attempt to download the
	full box prior to discovering this error.
	Downloading or copying the box...
	Extracting box...te: 4563k/s, Estimated time remaining: 0:00:01)
	Successfully added box 'opscode-ubuntu-1204' with provider 'virtualbox'!
	[default] Importing base box 'opscode-ubuntu-1204'...
	[default] Matching MAC address for NAT networking...
	[default] Setting the name of the VM...
	[default] Clearing any previously set forwarded ports...
	[default] Creating shared folders metadata...
	[default] Clearing any previously set network interfaces...
	[default] Preparing network interfaces based on configuration...
	[default] Forwarding ports...
	[default] -- 22 => 2222 (adapter 1)
	[default] Booting VM...
	[default] Waiting for VM to boot. This can take a few minutes.
	[default] VM booted and ready for use!
	[default] Configuring and enabling network interfaces...
	[default] Mounting shared folders...
	[default] -- /vagrant


!SLIDE commandline incremental transition=scrollVert
# Lo probamos usando Vagrant

Nos conectamos a la consola de la máquina creada y verificamos los usuarios
creados:

	$ vagrant ssh
	Welcome to Ubuntu 12.04.2 LTS (GNU/Linux 3.5.0-23-generic x86_64)

	 * Documentation:  https://help.ubuntu.com/
	Last login: Thu Jun  6 16:06:44 2013 from 10.0.2.2
	vagrant@ubuntu-12:~$ getent passwd 

!SLIDE smaller transition=scrollVert
# Lo probamos usando Vagrant

## Probemos la receta

Tenemos que editar el archivo Vagrantfile indicando que queremos aprovisionarlo
con **chef**. El archivo deberá quedar:

	@@@ ruby
	Vagrant.configure("2") do |config|
	  config.vm.box = "opscode-ubuntu-1204"
	  config.vm.box_url = "http://vagra..."
	  config.vm.network :private_network, ip: "33.33.33.100"
	  config.vm.provision :chef_solo do |chef|
	    chef.cookbooks_path = "./cookbooks"
	    chef.add_recipe "test"
	  end
	end

!SLIDE smaller transition=scrollVert
# Lo probamos usando Vagrant

## Probemos la receta

Una vez editado el Vagrantfile, reiniciamos vagrant y observamos la salida.

	$ vagrant reload
	[default] Attempting graceful shutdown of VM...
	....
	....
	[default] -- /vagrant
	[default] -- /tmp/vagrant-chef-1/chef-solo-1/cookbooks
	[default] Running provisioner: chef_solo...
	Generating chef JSON and uploading...
	Running chef-solo...
	stdin: is not a tty
	[2013-06-06T17:06:24+00:00] INFO: *** Chef 11.4.4 ***
	[2013-06-06T17:06:27+00:00] INFO: Setting the run_list to
	 ["recipe[test]"] from JSON
	[2013-06-06T17:06:27+00:00] INFO: Run List is [recipe[test]]
	[2013-06-06T17:06:27+00:00] INFO: Run List expands to [test]
	[2013-06-06T17:06:27+00:00] INFO: Starting Chef Run for
	ubuntu-12.04.2-cespi-amd64
	[2013-06-06T17:06:27+00:00] INFO: Running start handlers
	[2013-06-06T17:06:27+00:00] INFO: Start handlers complete.
	...
	...

