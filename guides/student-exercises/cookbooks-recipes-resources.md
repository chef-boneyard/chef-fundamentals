Cookbooks, Recipes, and Resources
======================

## Objectives

* Create a new cookbook
* Write a simple recipe with two resources
* Run Chef with the cookbook on a node

## Acceptance Criteria

* Directory `/var/www` is created and owned by www-data user.
* File `/var/www/index.html` is rendered from a template.
* Answer the questions.

## Create webserver Cookbook

Create a cookbok named `webserver` in the cookbooks directory. It
should have a `default.rb` recipe that configures two resources:

* The `/var/www` directory, which should be owned by the www-data
  user.
* An `index.html` file in `/var/www` rendered from a template, also
  owned by www-data.

The source template should contain information about the
node from attributes. Minimum:

* Platform and Platform version (`node['platform']`,
  `node['platform_version']`)
* Default IP address (`node['ipaddress']`)
* Fully qualified domain name (`node['fqdn']`)
* EC2 public IP address (`node['ec2']['public_ipv4']`)
* The node's run list (`node.run_list.to_s`)

Create the `metadata.rb` file with the cookbook's initial version. Use
any version number you like using the form X.Y.Z, e.g., "1.0.0" or
"0.5.2".

Upload the cookbook to the Chef Server.

__Note__: We are not installing Apache HTTPD as part of this
  exercise. The `index.html` file can be viewed as plain text.

## Apply webserver Cookbook to a node

Add the `webserver` cookbook's default recipe to your existing node
from the previous exercise using Knife.

Log into the node and run `chef-client`. Turn debug logging on to see
more information about the Chef run.

## Questions

What is the run list of the node?

What resources were created on the node?

What is the content of the file `/var/www/index.html`?
