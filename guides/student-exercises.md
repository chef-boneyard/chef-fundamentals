Getting Started
======================

## Objectives

* Install Ruby and Chef
* Create an initial Chef Repository
* Set up connectivity to Chef Server
* Questions

## Acceptance Criteria

* Executables chef-client, knife, ohai and shef are in the user PATH
* An initial Chef Repository is created (optionally under version control).
* Successful connection to the Chef Server can be made with `knife`.
* Answered the questions.

## Install Ruby and Chef

If Ruby and Chef are not already installed on the local workstation,
do so now. Use the Opscode Full Stack Installer:

* http://www.opscode.com/chef/install

__Hint__: This will by default install in `/opt/opscode` or
  `C:\Opscode` depending on platform.

## Create Chef Repository

Create a directory named `chef-repo` in your home directory on the
local workstation. It should have two sub-directories `.chef`, and
`cookbooks`.

### Optional:

Make the Chef Repository version controlled. You may use Git,
Subversion or your favorite version control system.

## Access Chef Server

Create the knife configuration file in the Chef Repository to connect
to the Chef Server. Copy your user private key file and Chef Server
validation key file to the same directory.

Verify connectivity to the Chef Server for your user with knife on the
local workstation.

## Questions

Where are Chef and Ruby installed on the system?


What version of Chef is installed? What command can be used to
determine this?


What version of Ruby is installed? What command can be used to
determine this?


What version of RubyGems is installed? What command can be used to
determine this?


What does `ohai` detect for the current system's platform and
platform_version?


What kinds of sub-commands are available to `knife`?


Where are the Chef command manual pages available on the local system, and what
formats are available?


Anatomy of a Chef Run
======================

## Objectives

## Acceptance Criteria

## Questions

Example Cookbook
======================

## Objectives

## Acceptance Criteria

## Questions

Roles
======================

## Objectives

## Acceptance Criteria

## Questions

Adding Cookbooks
======================

## Objectives

## Acceptance Criteria

## Questions

Node Attributes
======================

## Objectives

## Acceptance Criteria

## Questions

Introduction to Search
======================

## Objectives

## Acceptance Criteria

## Questions

Adding a New Node
======================

## Objectives

## Acceptance Criteria

## Questions

Data Bags
======================

## Objectives

## Acceptance Criteria

## Questions

Troubleshooting
======================

## Objectives

## Acceptance Criteria

## Questions

Chef Development
======================

## Objectives

## Acceptance Criteria

## Questions

