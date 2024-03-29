Upgrading
============

Redis Version Requirement
--------------

For those who are running Canvas with Redis enabled, the 2013-10-05 release will have a new minimum version requirement for redis: you'll need to be on redis 2.6 or higher. We are utilizing the new redis scripting commands that were added in 2.6. (https://groups.google.com/forum/#!topic/canvas-lms-users/oYpCmlWNtKM)

Steps to upgrade Redis:
```
#clear off the old one

apt-get --purge remove redis-server
apt-add-repository --remove ppa:instructure/backports

#add new redis 2.6 tracking repository

apt-add-repository ppa:chris-lea/redis-server
apt-get update
apt-get install redis-server
```

Canvas Upgrade
--------------

There are a number of things to keep in mind when upgrading the source code for Canvas.

For the most part, it is easiest to upgrade Canvas if you have kept your Canvas source code in a Git repository. Then, you can simply run 

```
git fetch && git reset --hard origin/stable
```

or some equivalent Git command.

Please note that this command will remove any staged or committed changes that you have made to your local repository. However, if you have a checkout of our stable branch from 2011-08-06 or later, you can also do

```
git fetch && git pull --rebase
```

to safely update your stable branch and keep your changes. Prior to 2011-08-06 we were force pushing divergent branches to stable (sorry) so `git pull --rebase` would give you conflicts.


Configuration and locally stored files
--------------

If you did not upgrade using Git, you will need to know what files to replace, and what files to keep. In general, you want to keep anything in the `config` directory that ends in `.yml`. Further, if you are using local storage for uploaded files and attachments, you will want to preserve files in your local storage directory, which defaults to `tmp/files`

Upgrade and Install Bundled Gems
-------------
After upgrading the source make sure to also upgrade all required gems:

```
$GEM_HOME/bin/bundle install
```

Upgrade and Install Node Modules
-------------
You also may need to upgrade or install new node modules

```
yarn install
```

Compiled Assets
-------------

Regardless of how you upgrade your Canvas installation, once you are done, you will need to recompile a number of assets, such as Javascript and CSS files. We provide a tool that can do this for you.

```
$GEM_HOME/bin/bundle exec rake canvas:compile_assets
```
And if you are in a production environment, you will need to specify that:

```
RAILS_ENV=production $GEM_HOME/bin/bundle exec rake canvas:compile_assets
```

Please note that compile_assets may take very long to run, and that at least 8 GB of RAM is recommended for running compile_assets in production.

Database migrations
-------------

Often, there is some minor database migration that needs to happen for all of the latest Canvas features to work. There is a built in Rails command to do this:

```
$GEM_HOME/bin/bundle exec rake db:migrate
```

If you are running a production environment, you will probably want to specify that. In addition, to ensure less downtime, you can run "predeploy" migrations (changes to schema required for the new version, but backwards compatible with old, currently running code) separately prior to rolling the code out to your full production environment.

```
<from a location with access to the production database, but not currently serving web requests or running jobs>
RAILS_ENV=production $GEM_HOME/bin/bundle exec rake db:migrate:predeploy
<deploy new code to running web servers and job servers>
RAILS_ENV=production $GEM_HOME/bin/bundle exec rake db:migrate
```

Notification types
-------------

Sometimes we add new notification types to Canvas. You can make sure yours are up to date by running:

```
$GEM_HOME/bin/bundle exec rake db:load_notifications
```

Again, if you are running a production environment, you will probably want to specify that.

```
RAILS_ENV=production $GEM_HOME/bin/bundle exec rake db:load_notifications
```
