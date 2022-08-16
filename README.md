# Con inline
0. Poner en línea de comando ´´´vagrant init´´´ esto creará un archivo llamado Vagrantfile.
1. Primero definiremos la version de linux a usar: en este caso será la ubuntu/jammy64.
2. Indicamos al Vagrantfile la version de linux con la provision de comandos para instalar nodejs.
´´´
apt-get update
apt-get upgrade
apt-get install -y nodejs 
apt-get install -y nodejs npm
cd app 
npm install
node app.js
´´´
3. El vagrantfile se verá así:
´´´
config.vm.box = "ubuntu/jammy64"
  config.vm.provision :file, source: './app', destination: "$HOME/app"
  config.vm.provision "shell",
    inline: "
    apt-get update
    apt-get upgrade
    apt-get install -y nodejs 
    apt-get install -y nodejs npm
    cd app 
    npm install
    node app.js
    "
´´´
4. Despues con el comando ´´´vagrant up´´´ levantamos la máquina y se empieza a bajar la imagen además de instalar nodejs. Despues de aproximadamente 10 minutos tendremos finalmente la máquina con nodejs instalado.
5. Al meter a la línea de comando ´´´vagrant ssh´´´ podremos meternos a la línea de comando de la máquina recien creada. Al ingresar ´´´node -v´´´ nos imprimirá la versión.
\\\\
![confirmación de funcionamiento](./figs/20220808203134.png)
![app corriendo](./figs/20220808221706.png)
6. Para correr solo se debe de poner 'vagrant up' y se correrá.


# Con ansible
0. Poner ´´´vagrant init´´´ y agregarle lo siguiente al archivo:
´´´
Vagrant.configure("2") do |config|
    config.vm.box = "ubuntu/kinetic64"
    # copiar la app de host a guest
    config.vm.provision :file, source: './app', destination: "$HOME/app"
    config.vm.provision "ansible" do |ansible|
      ansible.playbook = "playbook.yml"
    end
    config.vm.provider "virtualbox" do |vb|
      vb.customize [ "modifyvm", :id, "--uartmode1", "disconnected" ]
    end
    config.vm.provider :virtualbox do |vb|
      vb.gui = true
    end
  end
´´´

1. Crear un archivo llamado playbook.yaml y agregar los comandos correspondientes.
´´´
- hosts: all
  name: Ansible
  tasks:
    - name: Install required system packages
      apt:
        pkg:
          - apt-transport-https
          - ca-certificates
          - curl
          - software-properties-common
          - python3-pip
          - virtualenv
          - python3-setuptools
        state: latest
        update_cache: true
    - name: install fcntl
      pip:
        name: fcntl
    - name: update
      shell: 
        apt-get update
    - name: upgrade
      shell: 
        apt-get upgrade
    - name: install nodejs
      shell: 
        apt-get install -y nodejs 
    - name: install npm
      shell: 
        apt-get install -y nodejs npm
    - name: run app
      shell: 
        cd app && node app.js
´´´

2. Con ´´´vagrant up´´´ podemos verlo funcionar.
