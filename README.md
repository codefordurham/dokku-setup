Summary
=======

An [ansible](http://ansible.com) playbook to maintain a dokku instance on AWS for
[Code for Durham](http://codefordurham.com).

Setup
-----

```
mkvirtualenv -p `which python2` dokku-setup

pip install -r requirements.txt
ansible-galaxy install -r requirements.yml
```

NOTE: you will also need the vault password for some passwords stored in
ansible. Ask another developer for it.

Deployment
----------

Run the setup.yml ansible script and setup docker-machine to point to the new
EC2 instance:

```
ansible-playbook setup.yml

# If your AWS profile is not default, use:
# AWS_PROFILE=YOURAWSPROFILE ansible-playbook setup.yml
```

Setup
-----

[Install ansible](http://docs.ansible.com/ansible/intro_installation.html), and
install [ansible galaxy](https://galaxy.ansible.com/) dependencies:

```bash
sudo ansible-galaxy install -r requirements.yml
```

Create your `vars/aws_secrets.yml` file with your AWS credentials needed to
create Route53 entries:

```
rm vars/aws_secrets.yml
ansible-vault edit vars/aws_secrets.yml
```

The file should look like:

```
---
aws_access_key: YOURKEY
aws_secret_key: YOURSECRET
```

Modify the script settings in `vars/aws.yml` to match your particular
configuration.

Teardown
--------

To remove all the resources execute the teardown script:

```
ansible-playbook teardown.yml

# remove the docker-machine setting:
docker-machine rm ...
```

Note that this will not remove any DNS entries created by applications you
deployed (apart from the one created for the EC2 server itself). You will need
to manually remove those from Route53.
