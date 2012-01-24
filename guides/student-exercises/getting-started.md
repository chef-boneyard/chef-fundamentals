Getting Started
======================

## Objectives

* Install Ruby and Chef
* Create an initial Chef Repository
* Set up connectivity to Chef Server

## Acceptance Criteria

* Executables chef-client, knife, ohai and shef are in the user PATH
* An initial Chef Repository is created (optionally under version control).
* Successful connection to the Chef Server can be made with `knife`.
* Answer the questions.

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

__Note (Windows)__: Explorer cannot create a file starting with `.`,
use `cmd.exe` or `powershell.exe`.

### Optional:

Make the Chef Repository version controlled. You may use Git,
Subversion or any other version control system you wish.

## Create Opscode Hosted Chef Account

Create a user account for Opscode Hosted Chef, then sign in and create
an organization. Remember that the organization short name must be
ASCII alpha-numeric, no spaces (dashes and underscores **are** allowed).

* http://www.opscode.com/hosted-chef/ -> "Free Trial"

## Access Chef Server

Create the knife configuration file in the Chef Repository to connect
to the Chef Server. Copy your user private key file and Chef Server
validation key file to the same directory.

Verify connectivity to the Chef Server for your user with knife on the
local workstation.

## Questions

Where are Chef and Ruby binaries installed on the system?

What version of Chef is installed? What command can be used to
determine this?

What version of Ruby is installed? What command can be used to
determine this?

What version of RubyGems is installed? What command can be used to
determine this?

What does `ohai` detect for the current system's `platform` and `platform_version`?

What kinds of sub-commands are available to `knife`?

Where are the Chef command manual pages available on the local system,
and what formats are available?
