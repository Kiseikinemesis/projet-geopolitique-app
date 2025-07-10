Vagrant.configure("2") do |config|
  # VM Web
  config.vm.define "web" do |web|
    web.vm.box = "ubuntu/jammy64"
    web.vm.hostname = "web"
    web.vm.network "private_network", ip: "192.168.56.11"
    web.vm.network "forwarded_port", guest: 80, host: 8080

    web.vm.provider "virtualbox" do |vb|
      vb.name = "web"
      vb.memory = 1024
      vb.cpus = 1
    end
    web.vm.provision "shell", inline: <<-SHELL
      apt update
      apt install -y nginx python3 python3-pip python3-venv git

      systemctl enable nginx
      systemctl start nginx
      echo "<h1>It works! Nginx is running on web VM.</h1>" > /var/www/html/index.html
      # Copier le fichier de conf nginx depuis le dossier partagé /vagrant vers la conf nginx
      cp /vagrant/nginx.conf /etc/nginx/sites-available/default
      # Vérifier la syntaxe de la config nginx
      nginx -t
      # Redémarrer nginx pour prendre en compte la nouvelle config
      systemctl restart nginx

      # Créer un dossier pour l'application Flask
      mkdir -p /home/vagrant/myapp
      chown -R vagrant:vagrant /home/vagrant/myapp

      # Installer virtualenv, créer l'environnement et installer les dépendances
      apt install -y python3-venv

      # Créer le virtualenv dans /home/vagrant/venv si pas déjà présent
      if [ ! -d /home/vagrant/venv ]; then
        python3 -m venv /home/vagrant/venv
      fi

      # Installer les dépendances Python via pip dans le venv
      /home/vagrant/venv/bin/pip install --upgrade pip
      if [ -f /home/vagrant/myapp/requirements.txt ]; then
        /home/vagrant/venv/bin/pip install -r /home/vagrant/myapp/requirements.txt
      fi

      # Copier le fichier de service systemd depuis le dossier partagé (./myapp/myapp.service)
      cp /vagrant/myapp/myapp.service /etc/systemd/system/myapp.service

      # Recharger systemd et activer le service Gunicorn
      systemctl daemon-reload
      systemctl enable myapp.service
      systemctl start myapp.service
    SHELL

    web.vm.synced_folder "./myapp", "/home/vagrant/myapp"
  end

  # VM DB
  config.vm.define "db" do |db|
    db.vm.box = "ubuntu/jammy64"
    db.vm.network "private_network", ip: "192.168.56.13"
    db.vm.provider "virtualbox" do |vb|
      vb.name = "db"
      vb.memory = 1024
      vb.cpus = 1
    end
    db.vm.provision "shell", inline: <<-SHELL
      apt update
      apt install -y mysql-server
    SHELL
  end
end
