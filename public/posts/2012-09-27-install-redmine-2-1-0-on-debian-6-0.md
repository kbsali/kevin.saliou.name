Install Redmine 2.1.0 on Debian 6.0
===================================

Install necessary packages
--------------------------

    aptitude update
    aptitude install ruby rubygems libruby libapache2-mod-passenger libmysqlclient-dev libmagickcore-dev libmagickwand-dev mysql-server

Install Not So necessary packages
---------------------------------
    aptitude install vim git

Download redmine + untar + move it
----------------------------------
    wget http://rubyforge.org/frs/download.php/76448/redmine-2.1.0.tar.gz
    tar xzvf redmine-2.1.0.tar.gz
    mv redmine-2.1.0 /usr/local/share/redmine-2.1.0
    ln -s /usr/local/share/redmine-2.1.0 /usr/local/share/redmine
    chown -R root:root /usr/local/share/redmine-2.1.0

Create a redmine user
---------------------
    adduser redmine

Ruby/Gem thingies...
--------------------
    gem install bundler
    cd /usr/local/share/redmine
    /var/lib/gems/1.8/bin/bundle install --without development test postgresql sqlite

DB setup
--------
    mysql -uroot -p -e "create database redmine character set utf8;create user 'redmine'@'localhost' identified by 'redmine_password';grant all privileges on redmine.* to 'redmine'@'localhost';"
    cp config/database.yml.example config/database.yml
    vi config/database.yml

    .. --- config/database.yml ---
    production:
      adapter: mysql
      database: redmine
      host: localhost
      username: redmine
      password: redmine_password
    ..

    RAILS_ENV=production /var/lib/gems/1.8/bin/rake db:migrate
    RAILS_ENV=production /var/lib/gems/1.8/bin/rake redmine:load_default_data
    /var/lib/gems/1.8/bin/rake db:encrypt RAILS_ENV=production

Apache setup
------------
    echo '<VirtualHost *:80>
     ServerName redmine.example.com
     DocumentRoot /usr/local/share/redmine/public
     <Directory /usr/local/share/redmine/public>
       AllowOverride all
       Options -MultiViews
     </Directory>
    </VirtualHost>' > /etc/apache2/sites-available/redmine.example.com
    a2ensite redmine.example.com
    a2dissite 000-default
    service apache2 restart

Email setup (through Gmail)
---------------------------
    cp config/configuration.yml.example config/configuration.yml
    vi config/configuration.yml

    .. --- config/configuration.yml ---
    default:
    # Outgoing emails configuration (see examples above)
    email_delivery:
      delivery_method: :smtp
      smtp_settings:
        enable_starttls_auto: true
        address: "smtp.gmail.com"
        port: 587
        domain: "smtp.gmail.com"
        authentication: :plain
        user_name: "USERNAME@gmail.com"
        password: "PASSWORD"
    ..

Visit http://redmine.example.com/settings/edit page and modify "Host name and path" value (so the links published in the emails are correct)

Git + Github integration
------------------------

    gem install json

    # This is how to install a plugin!
    cd plugins
    git clone https://github.com/koppen/redmine_github_hook.git
    RAILS_ENV=production /var/lib/gems/1.8/bin/rake redmine:plugins:migrate

Check http://redmine.example.com/admin/plugins

In case you do not have SSH keys...
    ssh-keygen -t rsa -C "redmine@example.com"

Visit https://github.com/settings/ssh and add the newly generated public key (see https://help.github.com/articles/generating-ssh-keys)

    su redmine
    mkdir /PATH/TO/REPOS
    cd /PATH/TO/REPOS
    git clone --mirror git@github.com:USER/project.git
    cd project.git
    git fetch -q --all
    echo '*/5 * * * * redmine cd /PATH/TO/REPOS/project.git && git fetch -q --all' > /etc/cron.d/sync_git_repos

Note : here you will have to configure you *ssh keys* :

Add the repo to Redmine : http://redmine.adticket.de/projects/elvis/settings?tab=repositories

* Now on github :
  * Visit https://github.com/USER/project/admin/hooks
  * Add a new *WebHook URL* : redmine.example.com/github_hook (if your project ID in Redmine is NOT the same as the project name in github : redmine.example.com/github_hook?project_id=my_project)

* Access Redmine through http://redmine.example.com
* Default admin user is already created : admin:admin

References
----------
* [HowTo Install Redmine 210 on Debian Squeeze with Apache Passenger](http://www.redmine.org/projects/redmine/wiki/HowTo_Install_Redmine_210_on_Debian_Squeeze_with_Apache_Passenger)
* [HowTo Install Latest Redmine on Debian 6 (Squeeze, Ruby-on-Rails, Apache2 Passenger)](http://hodza.net/2012/03/15/howto-install-redmine-on-debian-6-squeeze-ruby-on-rails-apache2-passenger/)
* [HowTo keep in sync your git repository for redmine](http://www.redmine.org/projects/redmine/wiki/HowTo_keep_in_sync_your_git_repository_for_redmine)
* [Redmine Github Hook](https://github.com/koppen/redmine_github_hook)

Done!

Updates
-------

* [2012-10-02] replace cron + vhost setup from "vi XXX" to "echo 'YYY' > XXX" so a simple copy/paste is enough