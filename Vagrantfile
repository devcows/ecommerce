# -*- mode: ruby -*-
# vi: set ft=ruby :
ENV['VAGRANT_DEFAULT_PROVIDER'] = 'virtualbox'
ENV['RAILS_ENV'] = 'development' if ENV['RAILS_ENV'].nil? || ENV['RAILS_ENV'] == ''

puts "##### Environment => #{ENV['RAILS_ENV']}"

Vagrant.configure(2) do |config|
  config.vm.define 'main_app' do |app|
    # Setup resource requirements
    app.vm.provider 'virtualbox' do |vb|
      vb.gui = false
      vb.memory = 2048
      vb.cpus = 2
    end

    app.vm.network 'forwarded_port', guest: 1080, host: 11080 # Mailcatcher
    app.vm.network 'forwarded_port', guest: 3000, host: 13000 # Rails app
    app.vm.network 'forwarded_port', guest: 5432, host: 15432 # Postgresql

    app.vm.hostname = 'ylosix-vm'

    # Ubuntu
     app.vm.box = 'box-cutter/ubuntu1604'

    # Set environment variables and rvm project config files
    app.vm.provision "shell", privileged: false, inline: <<-SHELL
      source ~/.profile && [ -z "$DATABASE_URL" ] && echo "export DATABASE_URL=postgres://postgres:postgres@127.0.0.1:5432/ecommerce" >> ~/.profile
      source ~/.profile && [ -z "$RAILS_ENV" ] && echo "export RAILS_ENV=development" >> ~/.profile
      echo 2.3.0 > .ruby-version
      echo ylosix > .ruby-gemset
    SHELL
    # provision Docker and run postgres container
    config.vm.provision "docker" do |d|
      d.run "postgres:9.4.1",
        #cmd: "bash -l",
        #args: "-v '/vagrant:/var/www'"
        args: "-d -p 5432:5432 -v /vagrant:/vagrant -e 'POSTGRES_PASSWORD=postgres' --name postgres"
    end
    # install RVM
    app.vm.provision :shell, path: "vagrant/install-rvm.sh", args: "stable", privileged: false
    # install Ruby
    app.vm.provision :shell, path: "vagrant/install-ruby.sh", args: "2.3.0 rails bundler mailcatcher", privileged: false
    # Setup project dependencies and postgres container
    app.vm.provision 'shell', path: 'vagrant/setup-env.sh'
    app.vm.provision 'shell', path: 'vagrant/setup-app.sh', privileged: false
    # Launch app
    app.vm.provision 'shell', run: 'always', path: 'vagrant/start.sh', privileged: false
  end
end
