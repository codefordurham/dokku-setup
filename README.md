Summary
=======

An [ansible](http://ansible.com) playbook to maintain a dokku instance on AWS for
[Code for Durham](http://codefordurham.com).

Setup
-----

```
mkvirtualenv -p `which python3` dokku-setup

pip install -r requirements.txt
ansible-galaxy install -r requirements.yml
```

Setup
-----

Pre-requisites:

 * An EC2 Keypair: create one from the AWS console, and add it to vars/aws.yml
   -- share this with other developers that can setup dokku.
 * You will also need the vault password for some passwords stored in ansible.
   Ask another developer for it.

Run the setup.yml ansible script and setup docker-machine to point to the new
EC2 instance:

```
ansible-playbook setup.yml
```

Note: When cloudformation sets up your server the first time it will used the
'ubuntu' user to connect to the server. Afterward, your admin users will have
accounts, you should run this script with one of those user names.

```
ansible-playbook -u USERNAME_IN_AWS_YML setup.yml
```

You can modify the variables in `var/aws.yml` to access to the server, its
stack, and hostname.

When you run the playbook for an instance that already exists you will need to
supply a username that is configured to administer the server (`ansible-playbook
-u ADMINUSER setup.yml`), as the ubuntu account will not be accessible after the
initial setup.

Teardown
--------

To remove the server and all the resources execute the teardown script:

```
ansible-playbook teardown.yml
```

Plugins
-------

Plugins in use:

```
ssh copelco@cfd sudo dokku plugin:install https://github.com/michaelshobbs/dokku-logspout.git
ssh copelco@cfd sudo dokku plugin:install https://github.com/dokku/dokku-letsencrypt.git
ssh copelco@cfd sudo dokku plugin:install https://github.com/dokku/dokku-postgres.git postgres
ssh copelco@cfd sudo dokku plugin:install https://github.com/dokku/dokku-redis.git redis
ssh copelco@cfd sudo dokku plugin:install https://github.com/dokku/dokku-redirect.git
```

Dokku
-----

Once your dokku server is running, you can deploy apps to it. For example:

```
ssh ADMINUSER@DOKKU_HOST dokku apps:create python-sample

git clone https://github.com/heroku/python-sample.git
cd python-sample
git remote add dokku dokku@DOKKU_HOST:python-sample
git push dokku master
```
