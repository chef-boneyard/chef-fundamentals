Roles
======================

## Objectives

* Understand how to create roles with the Ruby DSL
* Upload a role to the Chef Server with knife
* Modify a node's run list directly with knife

## Acceptance Criteria

* Directory in Chef Repository for role Ruby DSL files
* `webserver` role containing `webserver`
* Target system's node uses the `webserver` role in its run list

## Create Webserver Role

Create a directory to store roles in the Chef Repository. It should be
named `roles`. Create a role named `webserver` as a Ruby DSL file in the
`roles` directory. Add the `webserver` cookbook to the role's run list.

Upload the `webserver` role to the Chef Server.

## Modify Node

Use `knife` to remove the `webserver` recipe from the node's run list
and add the `webserver` role.

Re-run `chef-client` on the node.

## Questions

What are the required components of a role?

How can the contents of the role on the Chef Server be displayed? What
command-line option will show the JSON source?

What `knife` command is used to display only the node's run list?

When `chef-client` is run again on the node, what is the content of
the `roles` attribute on the node object? What is the content of
`recipes`? What is the run list?

When `chef-client` is run again on the node, did it make any changes
to the managed resources?
