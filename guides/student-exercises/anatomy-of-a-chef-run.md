Anatomy of a Chef Run
======================

## Objectives

* Configure remote target to run `chef-client`
* Successful `chef-client` run with debug logging

## Acceptance Criteria

* Client and node objects exist on Chef Server
* Debug log output from `chef-client`
* Answer the questions

## Prepare Remote Target

**Ubuntu**

Your instructor will provide an IP address of an Ubuntu
system. Connect to the system with the following credentials:

* Username: ubuntu
* Password: opstrain_0150

**CentOS**

You should have the Virtual Machine created in VMware (Player,
Workstation, Fusion) on your workstation. The login credentials are:

* Username: root
* Password: opscodechef

Log in and run `/sbin/ifconfig` or `/sbin/ip addr` to get the system's
IP address.

**All**

You will need to install Chef on the system. Use the Full Stack
installer.

* http://opscode.com/chef/install

You may need to install the `curl` package:

    sudo apt-get install curl

## Create Configuration

Create a configuration file for your remote target system. The default
location per platform:

* Unix/Linux: `/etc/chef/client.rb`
* Windows: `C:\chef\client.rb`

At a minimum, the configuration should include the Chef Server URI
(`chef_server_url`), and the name of the validation API client
(`validation_client_name`). Refer to your Knife configuration file for
values of these settings. Also, set `node_name` to
`"webserver.localdomain"`.

## Use Validation Key

Use the validation key for the Chef Server to automatically create the
new API client. It should be copied to the same directory as the
`client.rb` file, and the filename should be `validation.pem`.

## Run Chef Client

Run `chef-client` on the local system with debug logging, and send the
output to a file. Use the output file, along with the command-line
tools to answer the following questions.

**Ubuntu**: You'll need to either su to root or use sudo to execute
`chef-client`.

## Questions

What is the name of the node and client created on the Chef Server?
What commands can be used to get these values?

What are two ways to change the name of the node and client at
`chef-client` run time?

What are the platform and platform version of the node?

Does the node have a run list?

What is the IP address detected for the node? Is it the correct
default IP address?

Is the API client an admin?

Does the validation key file still exist? Why?

What kind of HTTP request is made to save the node? When does this
occur?
