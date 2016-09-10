---
layout: post
title: '#hacking  Ansible'
date: 2014-10-17 21:02:43.000000000 -07:00
categories:
- Coding
tags: []
status: publish
type: post
published: true
---
Before you continue, let me congratulate myself. It’s been years since I’ve written a post. Hooray! Also, thank you to <a href="https://github.com/radamanthus/ansible-rails">Rad</a> for guiding me on my Ansible adventure. The ansible code I reference here are <a href="https://github.com/gregmoreno/ansible-rails">available in github</a>.

There are tons of posts about Ansible already so this is more about gotchas I learned along the way.

#Variables

Just like good coding, you should isolate things that will change. Store variables in vars/defaults.yml for things like initial database password, deployment directory, ruby version, and whatnot. I started out with using snake format but I later learned you can have a hierarchy for your variables as well, for example:

{% highlight yaml %}
app:
  name: google.app
{% endhighlight %}

Inside your tasks, you can reference it with {{ app.name }}. Of course, nothing prevents you from just simply naming your variable as app_name and use it as {{ app_name }}

#Hosts and groups

You can use ansible to work on 1 or more servers at a time. These servers are specified in the hosts file (see hosts in the example). You can also group servers by using [groupname]. In the example below, you can have ansible target qa only (using the —limit option) and it will update the 2 servers you listed under the [qa] group.

{% highlight yaml %}
[local]
localhost ansible_python_interpreter=/usr/local/bin/python

[www]
www.yourapp.com

[staging]
staging.yourapp.com

[qa]
qa1.yourapp.com
qa2.yourapp.com
{% endhighlight %}

# Use roles

Roles allow you to separate common tasks into sort of like modules giving you flexibility. Some use roles to group common tasks like ‘db’, ‘web’, etc. For starters like me and if you are playing with combining different software, I used roles to define specific software. I have a role named ‘mysql’, a role ‘nginx-puma’, or a role ‘nginx-passenger’. Over time, you may split the roles into functional distinctions like web, db, etc.

# Creating an EC2 instance
In my example, just update the variables below (included in vars/defaults.yml) to suit your requirements.

{% highlight yaml %}
ec2_keypair: yourapp-www
ec2_instance_type: t1.micro
ec2_security_group: web-security-group
ec2_region: us-west-1
ec2_zone: us-west-1c
ec2_image: ami-f1fdfeb4 # Ubuntu Server 14.04 LTS (PV) 64-bit
ec2_instance_count: 1
{% endhighlight %}

You need your AWS_ACCESS_KEY, AWS_SECRET_KEY, and PEM file to create an instance. Creating the AWS access keys requires access to the IAM (Identity and Access Management).

When you’re ready, run the command:

{% highlight bash %}
AWS_ACCESS_KEY="access" AWS_SECRET_KEY="secret" ansible-playbook -i hosts ec2.yml
{% endhighlight %}

I have set ec2.yml to run against ‘local’ group (specified in the ’hosts’ file) because you don’t have a host yet. The Ansible EC2 module is smart enough to know that you are still creating an instance at this point.

After running the playbook, notice that it creates a new entry in the [launched] group in your hosts file. The new entry points to the EC2 instance you just created. At this point, you now have a server to run the rest of the playbooks.

# Creating the deploy user

When the instance is created, it would create a new user. In my project where I use Ubuntu, it creates an ‘ubuntu’ user. In other cases, it might only create ‘root’ user. Regardless, your next step is to create a ‘deploy’ user because it is not a good idea to keep using root. You can change the name of the deploy user (see vars/defaults.yml) but I prefer using ‘deploy’ because it is also the default user in capistrano and I’m a Rails guy.

{% highlight bash %}
ansible-playbook -i hosts create-user.yml --user root --limit launched --private-key ~/.ssh/yourapp.pem
{% endhighlight %}

This playbook does several things:

* Creates the ‘deploy’ user.
* Adds your ssh key to /home/deploy/.ssh/authorized_keys
* Gives deploy sudo rights.
* Disables ssh and password access for root.

Note that I specified in the initial user ‘root’. Depending on the instance you just created it might be ‘ubuntu’ or some other initial user

# Creating an instance in other providers

In Digital Ocean (and other similar providers), you can create an instance using their admin interface. The initial user is ‘root’ and you can specify a password and/or public key. If you forgot the public key, you must add it before you continue.

{% highlight bash %}
scp ~/.ssh/id_rsa.pub root@yourapp.com:~/uploaded_key.pub
ssh root@staging.app.com
mkdir -m og-rwx .ssh
cat ~/uploaded_key.pub >> ~/.ssh/authorized_keys
{% endhighlight %}

# Bootstrapping
Now that you have your deploy user, just run the command below, take a coffee, and come back after 30 mins. That’s how awesome ansible is.

{% highlight bash %}
ansible-playbook -i hosts bootstrap.yml --limit launched
{% endhighlight %}

You may need to tweak the roles or update a specific server. For example, you’re testing an upgrade on a staging server. In that case, make sure you specify the corect host group in the —limit parameter

A few more things I learned with this exercise:

* I can set the file mode with mkdir.
* Use `bash —login` to load login profile. When I ssh using ‘deploy@domain‘, my ruby gets loaded correctly. However, when using ansible (which uses ssh), my ruby is not loaded and thus, you will get errors like 'bundle not found'.
* I’m torn between using system ruby and chruby. On one hand, I feel like there is no need for chruby because this is a server and no need to switch rubies from time-to-time. On the other hand, why not try it and see how it works.
