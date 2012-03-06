Description
===========

This guide describes requirements set up an environment that Chef
Fundamentals students use for hands on exercises.

Specifics are written for Amazon EC2. If another environment is
used, then adjust accordingly based on the requirements.

Base Operating System
=====================

The target systems used by the Chef Fundamentals exercises are assumed
to be Ubuntu Server 10.04 or higher.

Required Network Access
=======================

Students will configure systems that need to be accessed on the
public-facing IP address through the following TCP ports:

* 22
* 80 and 443
* 22002

Login Credentials
=================

Students will log into the systems with a common username and
password. For consistency, use the password "opstrain_0150".

EC2 Implementation
==================

## Create Security Group

Use the EC2 API tools to manage the `fnd` security group as described
below. If multiple Chef Fundamentals classes are happening at the same
time and using the same EC2 account, modify the security group name
accordingly. It may be desirable to use a different region, e.g. when
conducting a class in Europe or Asia use the appropriate
region. Append "`--region REGION`" to the commands.

    export labgroup=fnd
    ec2addgrp ${labgroup} -d ${labgroup}
    ec2auth ${labgroup} -P tcp -s 0.0.0.0/0 -p 22
    ec2auth ${labgroup} -P tcp -s 0.0.0.0/0 -p 443
    ec2auth ${labgroup} -P tcp -s 0.0.0.0/0 -p 22002
    ec2auth ${labgroup} -P tcp -s 0.0.0.0/0 -p 8080
    ec2auth ${labgroup} -P tcp -s 0.0.0.0/0 -p 80

## Bootstrap Templates

Use the bootstrap template found in `.chef/bootstrap/fnd3-lab.erb` to
create new instances with the login credentials set. For example, this
will configure the `ubuntu` user to log in with the password "opstrain_0150".

## Launch Instances

This step is used for the systems which will be used by students as
nodes to configure. For each student, execute the following `knife
ec2` command.

    knife ec2 server create \
      -f m1.small -I $lucid_small \
      -x ubuntu -d fnd3-lab -G ${labgroup}

If another region is required, append "`--region REGION`" and "`-Z
ZONE`" to specify the region and availability zone in that region.

Replace "$lucid_small" with the current AMI from Canonical's list of
AMIs for Ubuntu:

* http://uec-images.ubuntu.com/releases/10.04/release/
