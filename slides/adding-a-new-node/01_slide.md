# Adding a New Node

.notes These course materials are Copyright © 2010-2012 Opscode, Inc. All rights reserved.
This work is licensed under a Creative Commons Attribute Share Alike 3.0 United States License. To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/3.0/us; or send a letter to Creative Commons, 171 2nd Street, Suite 300, San Francisco, California, 94105, USA.

# Objectives

At completion of this unit you should...

* Be able to use knife to configure a new server.
* Understand how search is used to integrate systems together.

# Bootstrapping Chef

The process of installing and initially running Chef is referred to as
“Bootstrapping Chef”. The general steps are:

* Update the local system’s package cache (OS dependent).
* Install Ruby and RubyGems.
* Install Chef as a RubyGem.
* Copy the validation certificate file.
* Create a Chef client configuration pointing at the right Chef Server.
* Create a JSON file with the node’s initial Run List.
* Run Chef using the JSON file.

# Installing and Running Chef

    sudo su -
    apt-get update
    apt-get install -y ruby ruby1.8-dev build-essential \
      wget libruby-extras libruby1.8-extras
    cd /tmp
    wget http://production.cf.rubygems.org/rubygems/rubygems-1.3.7.tgz
    tar zxf rubygems-1.3.7.tgz
    cd rubygems-1.3.7
    ruby setup.rb --no-format-executable

    gem install ohai chef
    mkdir /etc/chef
    vi /etc/chef/validation.pem
    vi /etc/chef/client.rb
    vi /etc/chef/first-boot.json

    sudo chef-client -j /etc/chef/first-boot.json

# Bootstrap Sub-command

Knife bootstrap will run a single script on a target host.

Uses “bootstrap templates” by name. Several come with Chef.

These are typically used to install Chef on the remote system and tell Chef to run with a particular run list.

# Default Bootstrap Template

    if [ ! -f /usr/bin/chef-client ]; then
      apt-get update
      apt-get install -y ruby ruby1.8-dev build-essential wget libruby-extras libruby1.8-extras
      cd /tmp
      wget http://production.cf.rubygems.org/rubygems/rubygems-1.3.7.tgz
      tar zxf rubygems-1.3.7.tgz
      cd rubygems-1.3.7
      ruby setup.rb --no-format-executable
    fi
    gem install ohai --no-rdoc --no-ri --verbose
    gem install chef --no-rdoc --no-ri --verbose <%= bootstrap_version_string %>
    mkdir -p /etc/chef
    (
    cat <<'EOP'
    <%= validation_key %>
    EOP
    ) > /tmp/validation.pem
    awk NF /tmp/validation.pem > /etc/chef/validation.pem
    rm /tmp/validation.pem
    (
    cat <<'EOP'
    <%= config_content %>
    EOP
    ) > /etc/chef/client.rb
    (
    cat <<'EOP'
    <%= { "run_list" => @run_list }.to_json %>
    EOP
    ) > /etc/chef/first-boot.json
    <%= start_chef %>

# Default Bootstrap Template

[Default Bootstrap Template](https://github.com/opscode/chef/raw/master/chef/lib/chef/knife/bootstrap/ubuntu10.04-gems.erb)

# Knife Bootstrap Templates

The templates that come with Chef:

* archlinux-gems
* centos5-gems
* fedora13-gems
* ubuntu10.04-apt
* ubuntu10.04-gems

# Knife Bootstrap Customization

The bootstrap “templates” are really just shell scripts.

You can create your own bootstrap templates.

Use `-d` to specify the distribution style.

    ./.chef/bootstrap/DISTRO.erb
    $HOME/.chef/bootstrap/DISTRO.erb

# Knife Bootstrap

    (
    cat <<'EOP'
    <%= config_content %>
    EOP
    ) > /etc/chef/client.rb

Bootstrap scripts are processed as ERB templates.

“`config_content`” comes from Bootstrap Context, which is a shortcut to
using the Chef Config values from Knife.

# Knife Bootstrap Usage

    % knife bootstrap IPADDRESS -r 'role[webserver]' -x ubuntu -i ~/.ssh/ec2.pem

    % knife bootstrap IPADDRESS -r 'role[webserver]' -x ubuntu -i ~/.ssh/ec2.pem -d ubuntu10.04-mine-gems

# Integrate a New Node

Integrating a new node into the infrastructure is easy with knife bootstrap.

We’ll add a load balancer that will sit in front of the web server.

# Add Load Balancer

Presumably, the new system has been provisioned and has an OS installed on it, ready to SSH and be managed with Knife Bootstrap and Chef.

# Load Balancer Cookbook

The load balancer will need a cookbook to install and configure the load balancer software.

Haproxy is the load balancer software.

The cookbook is available from the Community Site.

A new role will be created that will be applied with the “knife bootstrap” command.

# Load Balancer Role

    @@@ruby
    name "lb"
    description "web server load balancer"
    run_list(
      "recipe[haproxy::app_lb]"
    )

The `haproxy::app_lb` recipe will perform a search for web servers.

# Bootstrap Load Balancer

    knife bootstrap IPADDRESS -r 'role[lb]' -x ubuntu --sudo

IPADDRESS is the IP of the target system. We can use FQDN here.

Default Ubuntu installs enable the “ubuntu” user to sudo.

# Haproxy Uses Search

The app_lb recipe in the haproxy cookbook uses search.

Any new nodes with “role[webserver]” will be automatically detected by
the load balancer whenever it runs Chef.

.notes When growing the infrastructure, new web servers can be created
with the “webserver” role. Then run chef-client on the load balancer
node and the new servers will be automatically added to the pool for
haproxy.

# Summary

You should now be able to...

* Use knife to configure a new server.
* Describe how search is used to integrate systems together.

# Lab Exercise

## Adding a New Node
