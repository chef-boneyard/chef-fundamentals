Adding Cookbooks
======================

## Objectives

* Download additional cookbooks from the Chef Community Site
* Update `base` role with additional cookbooks
* Create a new `webserver` cookbook
* Apply the `webserver` cookbook to the target system

## Acceptance Criteria

* `apache2`, `apt` and `chef-client` cookbooks should be in the Chef Repository and Chef Server
* New cookbooks should be applied with `base` role.
* A new cookbook named `webserver` should be in the Chef Repository and Chef Server
* A new role named `webserver` should apply the `apache2` and `webserver` cookbooks

## Download Additional Cookbooks

Download and extract the following cookbooks from the Chef Community Site.

* `apache2`
* `apt`
* `chef-client`

Extract these into the `cookbook_path` and upload them to the Chef Server.

Add `apt` and `chef-client` to the `base` role's run list. Don't do anything else with the `apache2` cookbook yet.

## Create webserver Cookbook

Create a new cookbook named `webserver` in the Chef Repository `cookbook_path`. It should have `metadata.rb`, a default recipe and a template.

The default recipe should contain the following:

* Include the `apache2` recipe.
* A template for `/var/www/index.html`. Create the source as `index.html.erb`. It should be owned by the `node['apache']['user']` attribute.

The metadata should declare a dependency on the `apache2` cookbook.

The index.html should display the following node information:

* Node's `name`
* `platform` attribute
* `platform_version` attribute
* `fqdn` attribute
* `ec2['public_hostname']` attribute
* Expanded list of recipes in the node's run list.

To display the recipes in the node's run list, use:

`node.recipes.join(", ")`

Format and display the HTML page any way you want, including wrapped in a `<pre>` block. The point is to write ERB, not HTML.

## Create webserver Role

Create a new `webserver` role Ruby DSL file in the `roles` directory. It should have the `webserver` cookbook in its run list. Upload the role to the Chef Server.

## Add webserver Role and Run Chef

Use `knife` to add the `webserver` role to the target node's run list. Log into the system and run `chef-client`. Verify that Chef runs successfully, and then examine the node object on the Chef Server. Navigate to the node's IP address in a web browser to view the content of the index.html file.

## Questions

What is the node's run list on the Chef Server? Does it have equivalent content in the rendered web page?


What is the content of the `recipes` attribute? Is this different than earlier exercises? If so, how?


Is the `apache2` cookbook in the node's run list or the `recipes` attribute? Why or why not?


View the `webserver` cookbook on the Chef Server. What are its dependencies?


View the `base` role on the Chef Server. What is its run list?


