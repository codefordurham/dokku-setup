Summary
=======

An [ansible](http://ansible.com) playbook to setup a
[rancheros](http://rancher.com) as a simple PAAS on AWS.

The script will setup a t2.micro ec2 instance, security group settings, a public IP
address, and configure [nginx-proxy](https://github.com/jwilder/nginx-proxy) and
[nginx-route53](https://github.com/hmalphettes/docker-route53-dyndns) docker
instances. The default settings for this script will fall under the
free tier, if your AWS account is eligible for them.

Once setup you can deploy with docker to the server, and get a DNS entry
configured for your app, that sits behind the nginx-proxy. The following
environmental variables can be used to configure DNS and nginx-proxy:

 * VIRTUAL_HOST: (required). The full DNS name that your docker instance should
   resolve to.
 * ZONE: (optional). If your DNS zone is not the domain (ie, at.example.com
   rather than example.com), then you must specify the zone.

Example Deployment
------------------

Run the setup.yml ansible script and setup docker-machine to point to the new
EC2 instance:

```
ansible-playbook -i ec2.py setup.yml

# If your AWS profile is not default, use:
# AWS_PROFILE=YOURAWSPROFILE ansible-playbook -i ec2.py setup.yml

# copy/paste command to setup docker-machine:
docker-machine create ...
```

Once your new PAAS system is setup you can connect to it, and run docker
instances (rerun the idempotent setup.yml script if your IP has changed to
access the docker ports, which are restricted to your IP):

```
eval "$(docker-machine env codefordurham)"

# start an example 'whoami' server with a custom DNS entry:
docker run -e VIRTUAL_HOST=whoami.a.willowdesk.info jwilder/whomai

# test it out:
curl -H "Host: whoami.local" whoami.a.willowdesk.info
```

Needless to say, this works for docker-compose configurations as well.

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
ansible-playbook -i ec2.py teardown.yml

# remove the docker-machine setting:
docker-machine rm ...
```

Note that this will not remove any DNS entries created by applications you
deployed (apart from the one created for the EC2 server itself). You will need
to manually remove those from Route53.
