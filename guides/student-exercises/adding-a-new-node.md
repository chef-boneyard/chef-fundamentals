Adding a New Node
======================

## Objectives

* Use `knife bootstrap` to set up a new node as a load balancer in front of the existing target node.

## Acceptance Criteria

* `haproxy` cookbook in repository.
* `load_balancer` role to apply `haproxy` recipe that uses Search.
* Second target system is bootstrapped with `knife bootstrap` and configured as a load balancer.
* Answered questions.

## Acquire Haproxy Cookbook

Download the `haproxy` cookbook from the Chef Community Site. We're going to use the `app_lb` recipe. Review the recipe, and the attributes. Upload the cookbook to the Chef Server.

## Create Load Balancer Role

Create a new role named `load_balancer`. It should use the `haproxy` cookbook's `app_lb` recipe. The `haproxy` cookbook uses the attribute `member_port` to specify what port the web servers are listening to. Set this attribute in the role to "80".

Upload the role to the Chef Server.

## Bootstrap new Node

Your instructor will provide a second target system's IP address. Use the `knife bootstrap` command to automatically set up the system with the `base` and `load_balancer` roles with Chef.

Once complete, navigate to the public IP address of the load balancer system in your web browser on port 80. Then navigate to the same IP on port 22002.

## Questions

