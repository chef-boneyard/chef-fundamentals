Node Attributes
======================

## Objectives

* Set attributes via cookbook or role
* Understand the attribute priority between cookbook and role attributes

## Acceptance Criteria

* Attributes file in `webserver` cookbook
* `default_attributes` entry in the `webserver` role
* Node has a value for the `webserver.origin` attribute on Chef Server.

## Cookbook Attributes

In the `webserver` cookbook, create a `default.rb` attributes file and set a `default` level node attribute, `node['webserver']['origin']`, with content like "This is from the cookbook attributes file."

Upload the cookbook to the Chef Server and run Chef on your target system. Show the attribute from the node object on the Chef Server.

## Role Attributes

Add the `node['webserver']['origin']` attribute in the `webserver` role at the default level, with content like "This is from the role." Upload the role to the Chef server and run Chef on your target system. Show the attribute from the node object on the Chef Server.

## Optional: Recipe Attributes

Optionally add the `node['webserver']['origin']` attribute in a recipe using a normal level attribute, with content like "This is from the recipe."

## Questions

What is the value of the `webserver.origin` attribute for the node on the Chef Server after first adding the attribute in the cookbook attributes file?


What is the value of the `webserver.origin` attribute for the node on the Chef Server after adding the attribute in the role?

(Optional) If you added the attribute in the recipe, what is the value of the `webserver.origin` attribute for the node on the Chef Server? How can the role be modified to override this value?
