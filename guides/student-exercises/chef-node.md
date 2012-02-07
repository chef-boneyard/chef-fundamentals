Chef Node
======================

## Objectives

* View the node object on the Chef Server with knife
* Create new attributes on the node with a cookbook and role.
* Use node attributes in a conditional in the recipe.

## Acceptance Criteria

* Create attributes on the node with an attributes file.
* Override the attributes with a role.
* Answer questions about the node.

## View Node Object

Use the `knife node show` command with various options to show the
target remote system's node object on the Chef Server.

## Cookbook Attributes

In the `webserver` cookbook, create a `default.rb` attributes file and
set a `default` level node attribute, `node['webserver']['origin']`,
with content like "This is from the cookbook attributes file."

Upload the cookbook to the Chef Server and run Chef on your target
system. Show the attribute from the node object on the Chef Server.

## Role Attributes

Add the `node['webserver']['origin']` attribute in the `webserver`
role at the default level (`default_attributes`), with content like
"This is from the role."  Upload the role to the Chef server and run
Chef on your target system. Show the attribute from the node object on
the Chef Server.

## Attributes in a Recipe

Create a conditional based on the node's platform to output a log
message with the `log` resource. Example syntax:

    if node['platform'] == "ubuntu"
      log "I'm on #{node['platform']}"
    end

_Optional_ add the `node['webserver']['origin']` attribute in a recipe
using a normal level attribute (`node.set`), with content like "This is from the
recipe."

## Quesetions

What is your target remote system's node name? FQDN? Are they
different?

Show the node's ec2 attributes (`-a ec2`). Which one is used as the node name?

What is the node's run list? Its environment?

Use `-Fj` to display the node object as JSON. Are attributes from Ohai
shown?

What is the value of the `webserver.origin` attribute for the node on
the Chef Server after first adding the attribute in the cookbook
attributes file?

What is the value of the `webserver.origin` attribute for the node on
the Chef Server after adding the attribute in the role?

(Optional) If you added the attribute in the recipe, what is the value
of the `webserver.origin` attribute for the node on the Chef Server?
How can the role be modified to override this value?
