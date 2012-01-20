# Node Attributes

.notes These course materials are Copyright © 2010-2012 Opscode, Inc. All rights reserved.
This work is licensed under a Creative Commons Attribute Share Alike 3.0 United States License. To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/3.0/us; or send a letter to Creative Commons, 171 2nd Street, Suite 300, San Francisco, California, 94105, USA.

# Objectives

At completion of this unit you should...

* Use attributes in a recipe or template.
* Be able to create a cookbook attributes file.
* Override a cookbook attribute setting with a role.
* Understand when to set node attributes in a recipe instead of an attributes file.

# Using Node Attributes

Node attributes can be used in recipes and templates.

Access node attributes like Ruby hash keys.

* `node[“fqdn”]`
* `node[“apache”][“dir”]`

Node attributes can come from a variety of places.

# Ohai: Automatic Attributes

Ohai determines many node attributes automatically for Chef.

Values such as `platform`, `platform_version`, `ipaddress`, `hostname` are determined by Ohai when Chef runs.

These cannot be overridden.

You can find the attributes of a node by running “ohai”.

# Attribute Conditionals

Attributes can be used to set up conditional checks using Ruby “if”, “unless” and “case” statements.

This is commonly done to match a node’s platform.

# Attribute Conditionals

    @@@ruby
    package "apache2" do
      case node["platform"]
      when "centos","redhat"
        package_name "httpd"
      when "debian","ubuntu"
        package_name "apache2"
      end
      action :install
    end

    case node["platform"]
    when "debian", "ubuntu"
      package "apache2-doc" do
        action :install
      end
    end

# Attributes in a String

    @@@ruby
    template "#{node["apache"]["dir"]}/ports.conf" do
      source "ports.conf.erb"
      group "root"
      owner "root"
      variables :apache_listen_ports => node["apache"]["listen_ports"]
      mode 0644
      notifies :restart, resources(:service => "apache2")
    end

Use Ruby’s #{} string interpolation to embed an attribute in a string.

# Variables in a Template

    @@@ruby
    template "#{node["apache"]["dir"]}/ports.conf" do
      source "ports.conf.erb"
      group "root"
      owner "root"
      variables :apache_listen_ports => node["apache"]["listen_ports"]
      mode 0644
      notifies :restart, resources(:service => "apache2")
    end

# Variables in a Template

    #This file generated via template by Chef.
    <% @apache_listen_ports.each do |port| -%>
    Listen <%= port %>
    NameVirtualHost *:<%= port %>

    <% end -%>

# Node Attributes in Templates


    ServerRoot "<%= node["apache"]["dir"] %>"

    # ...

    <% if node["platform"] == "debian" || node["platform"] == "ubuntu" -%>

Node attributes used in a template can come from the cookbook itself, or one auto detected by chef such as “platform”.

# Cookbook Attributes

Node attributes can come from cookbooks.

Use an attributes file in a cookbook to set defaults.

These default values are to ensure that some value is set.

All cookbooks’ attributes files are loaded in alphabetical order by cookbook name.

# cookbooks/apache2/attributes/default.rb

    @@@ruby
    default["apache"]["listen_ports"] = [ "80","443" ]
    default["apache"]["contact"] = "ops@example.com"
    default["apache"]["timeout"] = 300

Use the “default” keyword to set the attributes.

The node object is implied, `default` is a method of Chef::Node.

# Attributes in Roles

Node attributes can be set in roles using “`default_attributes`”.

This overrides the attributes set using “default” in the cookbook’s attributes file.

# roles/webserver.rb

    @@@ruby
    name “webserver”
    description “Systems that serve HTTP traffic”
    run_list(
      “recipe[apache2]”,
      “recipe[apache2::mod_ssl]”
    )
    default_attributes(
      “apache” => {
        “listen_ports” => [ 80, 443, 8080 ]
      }
    )

# Attributes in Recipes

Setting attributes is the same as calling a method on the node object,
and this can also be done in a recipe.

Do this when you want to calculate a value for the attribute.

Calculated values can come from Ruby expressions, searches, external
data sources and more.

# Attributes in Recipes

When setting attributes in recipes, use the node.set method.

This allows additional overrides to be used, but has higher priority
than cookbook attributes files “default” and “set” and role
“`default_attributes`” values.

# Attributes in Recipes

    @@@ruby
    node.set['unicorn']['workers'] = [node['cpu']['total'].to_i * 4, 8].min

# Summary

You should now be able to...

* Create a cookbook attributes file.
* Use attributes in a recipe or template.
* Override a cookbook attribute setting with a role.
* Describe when to set attributes in a recipe instead of an attributes file.

# Additional Resources

* http://wiki.opscode.com/display/chef/Attributes

# Lab Exercise

## Node Attributes
