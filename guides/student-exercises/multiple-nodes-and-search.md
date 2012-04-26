Multiple Nodes And Search
======================

## Objectives

* Add load balancer role with haproxy recipe
* Bootstrap a new node with load balancer role
* Use knife ssh to rerun chef client

## Acceptance Criteria

* Second instance bootstrapped with `knife bootstrap`.
* Second instance running haproxy load balancing webserver.
* Answer questions

## Haproxy Cookbook

Unless you did so in the previous exercise, download the `haproxy`
cookbook from the Chef Community Site.

We're going to use the `app_lb` recipe. Review the recipe, and the
attributes. Upload the cookbook to the Chef Server.

## Create Load Balancer Role

Create a role `load_balancer.rb` in the roles directory. Use
`recipe[haproxy::app_lb]` as the only item in the node's run list.

The `haproxy` cookbook uses the attribute `member_port` to specify
what port the web servers are listening to. Set this attribute in the
role to "80".

Upload the role to the Chef Server.

## Knife bootstrap Second Target

If Chef is not version 0.10.10 or higher, retrieve the chef-full
bootstrap template and save it as `.chef/bootstrap/chef-full.erb` in
your Chef Repository.

https://raw.github.com/opscode/chef/master/chef/lib/chef/knife/bootstrap/chef-full.erb

Your instructor will provide a second target system's IP address or
you will use a second Virtual Machine. Use the `knife bootstrap`
command to automatically set up the system with the `base` and
`load_balancer` roles with Chef. When running the `knife bootstrap`
command, specify the new system's node name with `-N NODENAME`, e.g.
`-N load-balancer.localdomain`.

Once complete, navigate to the public IP address of the load balancer
system in your web browser on port 80. Then navigate to the same IP on
port 22002.

Use the `chef-full` distro bootstrap with knife bootstrap to set up
the load balancer.

The node's run list should contain: `role[base]` and
`role[load_balancer]`. Base should be applied first.

## Use Knife Search

Use knife search to answer some of the questions below. Queries to use:

* "role:webserver"
* "role:base"
* "platform:ubuntu"
* "recipes:apt"
* "role:load_balancer"

## Knife SSH

Use `knife ssh` to run `sudo chef-client` on all nodes with the role
`base` applied directly to the run list.

Use `knife ssh` to report the `uptime` of all nodes with the recipe
`webserver` appearing anywhere in the run list (`recipes:webserver`).

## Questions

How does the `app_lb` recipe in the haproxy cookbook differ from the
default recipe?

How does the `haproxy::app_lb` recipe determine what nodes to search?
Can it be modified? If so, where would that be done?

How many nodes have HTTP traffic balanced by haproxy?

What other attributes does the haproxy cookbook have?

The haproxy cookbook has a `default` recipe. How does it differ from
the `app_lb` recipe?

How many nodes are running Ubuntu?

What is the run list of all the nodes? What option shows only the run
list from node results?

What is the node name of systems that are webservers?

Do any roles have a recipe from the apache2 cookbook in their run
list?

Do any nodes have a recipe from the apache2 cookbook?
