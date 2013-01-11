####### Sharing for Good Application on AWS-ec2 server using Webistrano ########

Setup To Deploy Application

prerequisites: 1)  Installing  Rails without RVM

Install dependencies for Ruby 3.2.x

        $ sudo apt-get install libyaml-dev libxml2-dev libxslt1-dev  zlib1g-dev build-essential openssl libssl-dev libmysqlclient-dev libreadline6-dev
       
2)  Install ruby  from source file
        cd /usr/src
        $sudo wget http://ftp.ruby-lang.org/pub/ruby/1.9/ruby-1.9.3-p0.tar.gz
        $sudo tar xvzf ruby-1.9.3-p0.tar.gz
        cd ruby-1.9.3-p0
        $sudo ./configure --prefix=/usr/local --enable-shared --disable-install-doc --with-opt-dir=/usr/local/lib
        $sudo make
        $sudo make install     
        
3)  Test proper ruby is installed
        $sudo ruby -v
The above command should display some thing as follows
        ruby 1.9.3p194 (2012-04-20 revision 35410) [i686-linux]                
        
4)  Install ruby gem
        $cd /usr/src 
        $sudo wget http://production.cf.rubygems.org/rubygems/rubygems-1.8.24.tgz
        $sudo tar xvzf rubygems-1.8.24.tgz
        $cd rubygems-1.8.24
        $sudo ruby setup.rb        


5)  Install Mysql server and Mysql client
        $sudo apt-get install mysql-server-5.5 mysql-client-5.5

6)  Install rails
        $sudo gem install rails

7)  Install apache2
        $sudo apt-get install apache2
        
8)  Install passenger
        $cd /usr/src 
        $sudo wget http://rubyforge.org/frs/download.php/75548/passenger-3.0.11.tar.gz
        $sudo tar xvzf  passenger-3.0.11.tar.gz
        $cd passenger-3.0.11
        $sudo ./bin/passenger-install-apache2-module  

Follow the instruction provided by the installer and restart the apache

9)  Install Node.js 
        $sudo apt-get install python-software-properties
        $sudo add-apt-repository ppa:chris-lea/node.js
        $sudo apt-get update
        $sudo apt-get install nodejs-dev
        
The above all are require to deploy an Ruby application on Amazon Web Server using Webistrano.
Now we are moving towards configuration of the application in webistrano.

10)  Configure GitHub on server:
        $ sudo apt-get install git-core
authenticate server with github repository using ssh key, genrate and add into github account.
        $ ssh-keygen
then,add it to the GitHub account.
                
11) First we require an account on the AWS with ec2 instance to host an application on it.

Setting up an Amazon account
You can associate your new EC2 account with an existing Amazon account (if you already have one), or create a new account.

    Go to http://aws.amazon.com, and select Sign-up Now. Sign in to your existing Amazon account or create a new one.
    Go to http://aws.amazon.com/ec2, and select "Sign Up for Amazon EC2".
        Enter your credit card information.
        Complete your signup for the Amazon EC2 service. 

After signing up, you should end up at the EC2 console
        Create a key pair and download the private key
        Click Key Pairs under Networking and Security in the Navigation pane and then click the Create Key Pair button (save it in e.g. ~/.ec2/ec2.pem). 
        This  private key is for making SSH connections to newly created instances. 
        You will also need to set up your Amazon API credentials. Go to Account->Security Credentials
            click X.509 Certificates tab
            Create a new Certificate
            Download the private key and the certificate (save them in e.g. ~/.ec2/cert-XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX.pem and ~/.ec2/pk-             
            XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX.pem).
            Make your credential files private: chmod go-rwx ~/.ec2/*.pem
            Scroll to the bottom of the page and note your account ID (a number of the form XXXX-XXXX-XXXX). 

If at a later time you discover you need to generate a new X.509 certificate, click on "Your Account" at the top of the EC2 console page. You may need to click the small button with two down arrows near the top right of the EC2 console page to make the "Your Account" link visible. Then in the "Access Credentials" box, click the tab named "X.509 Certificates" and click "Create a New Certificate". Download the private key and certificate when prompted. 
  
12)  Login to the server(AWS-ec2 server):
        $ ssh -i <location_of_key/***.pem> username@server_ip
        
13)  Capistrano configuration using Webistrano:
Webistrano is a Web UI for managing Capistrano deployments. It lets you manage projects and their stages like test, production, and staging with different settings. Those stages can then be deployed with Capistrano through Webistrano. 

a)Open the the url "http://snakes.weboniselab.com:8000" and then signin to your account.
           
b)Create new project:
        Project: S4G
        Project Type: rails 
        
        Project configuration:
        Name 	                Value 	
        application 	        s4g 	                        // application name	
        deploy_to 	        /var/www/app/s4g 		// application deployment path
        deploy_via 	        :checkout 		
        rails_env 	        production 		        // Rails environment
        repository 	        git@github.com:webonise/s4g.git // Github repository ssh key
        scm 	                git 		                
        ssh_auth_methods 	publickey 	                // ssh authentication method
        use_sudo 	        false                           

c)Create Stage:
        Name 	                Value 	
        application 	        prod_s4g 	                //Stage name
        branch 	                feature 		        //Github branch name
        deploy_to 	        /var/www/app/S4G 		//application deployment path
        rails_env 	        production 		        //Rails environment
        runner 	                ubuntu 		                
        ssh_auth_methods 	publickey 		        //
        ssh_keys 	        ["/opt/bigthink.pem"] 		
        user 	                ubuntu                          //deploying user
        
d)Used recipes:
        Name 	                Description
        Bundle_install 	        Bundle install after update_code
        custom_database_yml 	Create symlink after bundle install
        custom_db_migrate 	Migrate database after create custom_symlink_log
        custom_symlink_log 	Symlink of server logs & newrelic.yml with shared
Follwing are the tasks:
        
        task :Bundleinstall do
                desc "Bundleinstall"
                run "cd  #{current_path}/ &&   bundle install  && bundle install --path vendor/cache" 
        end

        task :custom_symlink_log do
                desc "custom symlink log"
                run "rm -rf #{current_release}/log"
                run "ln -s #{shared_path}/log #{current_release}/log"
    
        # Create symlink of newrelic.yml file from shared to current release  
                run "ln -snf #{shared_path}/config/newrelic.yml #{current_release}/config/newrelic.yml"
        end 

        task :copy_database_yml do
                run "ln -s #{shared_path}/config/database.yml #{release_path}/config/database.yml"

                #run "chmod -R 777 #{release_path}/tmp" 
                #run "chmod -R 777 #{release_path} "
                # run " /etc/init.d/apache2 reload "
        end
  
        task :db_migrate do
                desc "db_migrate"
                run "cd #{current_release} && rake assets:clean"
                run "cd #{current_release} && rake assets:precompile"
                run "cd #{current_release} && rake db:migrate RAILS_ENV=production" 
        end 

        after "deploy:update_code", "Bundleinstall"
        after 'Bundleinstall', 'copy_database_yml'
        after 'copy_database_yml', 'custom_symlink_log'
        after "custom_symlink_log", "db_migrate"
  
e)  Deployed hosts: 
        Host 	        Role 	SSH Port 	Attributes 	        Status
        54.235.221.105 	app 	default 	 {:primary=>true} 	deployed 	
        54.235.221.105 	db 	default 	 {:primary=>true} 	deployed 	
        54.235.221.105 	web 	         default 		                deployed 	
        
NOTE: Please make sure that all the directories and file which are created by symlinks, having the writeable permission.    
    
After making the configuration the Export Capfile will look like:

        load 'deploy'
        # ================================================================
        # ROLES
        # ================================================================


            role :app, "54.235.221.105", {:primary=>true}
          
            role :db, "54.235.221.105", {:primary=>true}
          
            role :web, "54.235.221.105"
          

        # ================================================================
        # VARIABLES
        # ================================================================

        # Webistrano defaults
          set :webistrano_project, "s4_g"
          set :webistrano_stage, "prod_s4g"


          set :application, "prod_s4g"

          set :branch, "feature"

          set :deploy_to, "/var/www/app/S4G"

          set :deploy_via, :checkout

          set :rails_env, "production"

          set :repository, "git@github.com:webonise/s4g.git"

          set :runner, "ubuntu"

          set :scm, "git"

          set :ssh_auth_methods, "publickey"

          set :ssh_keys, ["/opt/bigthink.pem"]

          set :use_sudo, false

          set :user, "ubuntu"




        # ================================================================
        # TEMPLATE TASKS
        # ================================================================

                # allocate a pty by default as some systems have problems without
                default_run_options[:pty] = true
              
                # set Net::SSH ssh options through normal variables
                # at the moment only one SSH key is supported as arrays are not
                # parsed correctly by Webistrano::Deployer.type_cast (they end up as strings)
        #        [:ssh_port, :ssh_keys].each do |ssh_opt|
                  [:ssh_port, :ssh_keys, :ssh_auth_methods].each do |ssh_opt|
                  if exists? ssh_opt
                    logger.important("SSH options: setting #{ssh_opt} to: #{fetch(ssh_opt)}")
                    ssh_options[ssh_opt.to_s.gsub(/ssh_/, '').to_sym] = fetch(ssh_opt)
                  end
                end
                 


        # ================================================================
        # CUSTOM RECIPES
        # ================================================================


            task :copy_database_yml do
             run "ln -s #{shared_path}/config/database.yml #{release_path}/config/database.yml"                                 //create symlink of database.yml

             #run "chmod -R 777 #{release_path}/tmp" 
             #run "chmod -R 777 #{release_path} "
            # run " /etc/init.d/apache2 reload "

          end

        after 'Bundleinstall', 'copy_database_yml'

          task :Bundleinstall do
         desc "Bundleinstall"
             run "cd  #{current_path}/ &&   bundle install  && bundle install --path vendor/cache"                                     //Bundle Install 
         end

        after "deploy:update_code", "Bundleinstall"
          

          after 'copy_database_yml', 'custom_symlink_log'

        task :custom_symlink_log do
             desc "custom symlink log"
              run "rm -rf #{current_release}/log"                                                                                                      //remove log directory from release
              run "ln -s #{shared_path}/log #{current_release}/log"                                                                     //create symlink of logs
              
            # Create symlink of newrelic.yml file from shared to current release                                
              run "ln -snf #{shared_path}/config/newrelic.yml #{current_release}/config/newrelic.yml"           //create symlink of newrelic.yml
            
            # Create symlink of user images                                
              run "ln -snf #{shared_path}/public/uploads #{current_release}/public/uploads"           //create symlink of user images
        end

          after "custom_symlink_log", "db_migrate"

        task :db_migrate do
             desc "db_migrate"
             run "cd #{current_release} && rake assets:clean"                    //Remove compiled assets
             run "cd #{current_release} && rake assets:precompile"               //Compile all the assets named
             run "cd #{current_release} && rake db:migrate RAILS_ENV=production" //Migrate the databases
        end 

#================================================================================================================================================================

14)  Apache2 Configuration:

Make entry in /etc/apache2/httpd.conf
        
        <VirtualHost *:80>
                ServerName 54.235.221.105                        //server ip address
                #RailsAutoDetect off
                RailsEnv production                              //rails environment
                DocumentRoot /var/www/app/S4G/current/public            
          <Directory "/var/www/app/S4G/current/public">
                AllowOverride all
                Allow from all
          </Directory>
        </VirtualHost>

Now we need to restart the apache2 
        $ sudo /etc/init.d/apache2 restart     
        
        
Now,check the application on 'http://54.235.221.105'
        
       
 


        
        

