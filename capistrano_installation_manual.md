**************Deployment Of ROR application with Capistrano By Using Remote Database Server****************

Steps To Deploy Application:

Prerequisites : 1. Ruby Installation Firstly install rvm using following command

Install dependencies

    $ sudo apt-get install curl

Installing Ruby version manager(RVM)
 
    $bash < <(curl -s https://rvm.beginrescueend.com/install/rvm
    
append the following line in ~/.bashrc file

$  vim ~/.bashrc

    [[ -s "$HOME/.rvm/scripts/rvm" ]] && source "$HOME/.rvm/scripts/rvm"    
:wq!

Installing Ruby

    $ rvm install 1.9.2

To check ruby version

    $ ruby -v

Rails Installation To install rails issue following command

    $ gem install rails

Installing Capistrano To install Capistrano

    $ gem install Capistrano

Installing & Configuring Mysql To install mysql issue following command

    $ apt-get install mysql-server mysql-client

Installing Apache Server To install apache server issue following command

    $ sudo apt-get install apache2

Installation Procedure:

1) Create one ROR application and make it able to work simply with Github repository.

2) Then we are start working for deploy application with Capistrano by accessing the Remote Database server.

3) Install the Capistrano gem on the system,here we install Capistrano (2.13.5) 
    $ gem install capistrano

4) Installing CAPISTRANO.

I am installed capistrano-2.13.5, it is gem install.

    $ gem install capistrano

Successfully installed capistrano-2.13.5
1 gem installed
Installing ri documentation for capistrano-2.13.5...
Installing RDoc documentation for capistrano-2.13.5...

5) Edit config/databases.yml file & migrate database

   Before database Creation & Migration databses in mysql
   Edit config/database.yml

       development:
         adapter: mysql2
         encoding: utf8
         reconnect: false
         host: localhost
         database: demo_app_development
         pool: 5
         username: root
         password: password
         socket: /var/run/mysqld/mysqld.sock

Warning: The database defined as "test" will be erased and re-generated from your development database when you run "rake".

Do not set this db to the same as development or production.

Warning: The database defined as "test" will be erased and re-generated from your development database when you run "rake".

Do not set this db to the same as development or production.

       test:
         adapter: mysql2
         encoding: utf8
         reconnect: false
         host: localhost
         database: demo_app_test
         pool: 5
         username: root
         password: password
         socket: /var/run/mysqld/mysqld.sock

Same for Production environment

Before database migration create databses in mysql

    $bundle exec rake db:create

Now migrate database

Before database migration create databses in mysql

    $bundle exec rake db:create

Now migrate database
    $ bundle exec rake db:migrate

6) Test application locally

        $ rails s                                         //In project root directory
open given url in browser
        http://localhost:3000

Deploy Application on server using capistrano

We already done with the capistrano installation in Software installation section

Now capify your project goto your project folder

    $ cd /path to project directory/

then,

       $ capify .

This cmd creates Capfile & config/deploy.rb file

Authorize mysql user & Allow remote login

login to your mysql 

       $ mysql -u root -ppassword apply privileges to user
       GRANT ALL PRIVILEGES ON 'databasename'.* TO 'mysqluser'@'%' WITH GRANT OPTION;

change bind-address value of mysql configuration to your mysql database server

Now capify your project
goto your project folder
       $ cd /path to project directory/    
then,   
       $ capify .  
This cmd creates Capfile & config/deploy.rb file

Authorize mysql user & Allow remote login

login to your mysql 

       $ mysql -u root -ppassword apply privileges to user
       GRANT ALL PRIVILEGES ON 'databasename'.* TO 'mysqluser'@'%' WITH GRANT OPTION; 
       
change bind-address value of mysql configuration to your mysql database server

       $ sudo vim /etc/mysql/my.cnf 
       bind-address = 192.168.0.27 //Your database server ip 
  :wq!

Change value of the host: attribute to your webserver in config/database.yml file

       $ vim config/database.yml
       development:
        adapter: mysql2
        encoding: utf8
        reconnect: false
       host: 192.168.0.27                              //Database server ip
       database: sample_app_development
       pool: 5
       username: root
       password: admin
       socket: /var/run/mysqld/mysqld.sock

Warning: The database defined as "test" will be erased and re-generated from your development database when you run "rake". Do not set this db to the same as development or production.

       test:
         adapter: mysql2
         encoding: utf8 
         reconnect: false
         host: 192.168.0.27                                  //Database server ip
         database: sample_app_test
         pool: 5
         username: root
         password: admin
         socket: /var/run/mysqld/mysqld.sock

***** same for production environment*****

Dont forget to restart/reload the mysql service

       $ sudo /etc/init.d/mysql restart

Writing deploy.rb file

       $vim /path ot project dir/config/deploy.rb

       set :application, "Deployment demo" 
       set :scm, 'git' set :repository, "https://github.com/amolkhanorkar-webonise/capistrano_project_2" 
       set :branch, 'develop'

Repository branch

       set :use_sudo, false

Deployment directory on the server

       set :deploy_to, "/var/www/demo_app"

Server information

       role :web, "192.168.0.27"                   # Your HTTP server, Apache/etc
       role :app, "192.168.0.27"                    # This may be the same as your `Web` server
       role :db, “192.168.0.29”, primary => true     # Your Database server if you want to clean up old releases on each deploy uncomment this:
       after "deploy:restart", "deploy:cleanup"
       after "deploy:update_code", "assets:create_symlink"
       after "deploy:restart", “deploy:clean”  #Maintain recent five release
        namespace :assets do
        task :create_symlink do
       run "ln -s #{shared_path}/assets #{latest_release}/public/assets"
       run "cd #{latest_release} && bundle exec rake assets:precompile"
       run "ln -s #{shared_path}/config/database.yml #{current_path}/configdatabase.yml"
       end
    end

  :wq!

Deploy on server & configure virtualhost in apache

To deploy issue following command

       $ cap deploy

Note: If above command generates any error, troubleshoot it & reploy

Configure virtual host

Before that make sure Virtual hosting is enabled, To do this

       $ vim /etc/apache2/ports.conf
       NameVirtualHost  192.168.0.27
.
.
:wq!

Write VirtualHost Block

       $ vim /etc/apache2/httpd.conf
    
      <VirtualHost 192.168.0.27:80>
         ServerName local.sampleapp.com
         DocumentRoot  /var/www/capistrano_project_2/public/
         Rails_Env   development
      </VirtualHost>

   :wq!

Restart Apache2

       $sudo /etc/init.d/apache2 restart

Enjoy the Application !!

