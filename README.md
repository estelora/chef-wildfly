# Wildfly Cookbook
Cookbook to deploy Wildfly Java Application Server


[![Cookbook](http://img.shields.io/cookbook/v/wildfly.svg)](https://github.com/bdwyertech/chef-wildfly)
[![Build Status](https://travis-ci.org/bdwyertech/chef-wildfly.svg)](https://travis-ci.org/bdwyertech/chef-wildfly)
[![Gitter chat](https://img.shields.io/badge/Gitter-bdwyertech%2Fwildfly-brightgreen.svg)](https://gitter.im/bdwyertech/chef-wildfly)

# Updates 
- Optimized for Centos 6.6

# Requirements
- Chef Client 11+
- Java Cookbook (ignored if node['wildfly']['install_java'] is false)

# Platform
- CentOS, Red Hat

Tested on:
- CentOS 6.6
- Chef 12.12.15

# Usage
- I suggest you add users as a role attribute. However, you can customize them in attributes/users.rb
- I also suggest you set the Java attributes at the role level as well.


If running in production, I STRONGLY recommend you use a wrapper cookbook, and manually specify the Wildfly version, 
Java version (if node['wildfly']['install_java'] is true), and cookbook version as well.  (In the role).
This cookbook and configuration templates will continually be updated to support the latest stable release of Wildfly.  
Currently, version upgrades will trigger configuration enforcement, meaning any changes made outside of Chef will be wiped out.

# Attributes
* `node['wildfly']['install_java']` - Install Java using Java Cookbook.  Default `true`
* `node['wildfly']['base']` - Base directory to run Wildfly from

* `node['wildfly']['version']` - Specify the version of Wildfly
* `node['wildfly']['url']` - URL to Wildfly tarball
* `node['wildfly']['checksum']` - SHA256 hash of said tarball

* `node['wildfly']['user']` - User to run Wildfly as. DO NOT MODIFY AFTER INSTALLATION!!!
* `node['wildfly']['group']` - Group which owns Wildfly directories
* `node['wildfly']['server']` - Name of service and init.d script for daemonizing

* `node['wildfly']['mysql']['enabled']` - Boolean indicating Connector/J support

* `node['wildfly']['int'][*]` - Various hashes for setting interface & port bindings

* `node['wildfly']['smtp']['host']` - SMTP Destination host
* `node['wildfly']['smtp']['port']` - SMTP Destination port


# Recipes
* `::default` - Installs Java (if node['wildfly']['install_java'] is true) & Wildfly.  
Also installs Connector/J if you've got it enabled.
* `::install` - Installs Wildfly.
* `::mysql_connector` - Installs Connector/J into appropriate Wildfly directory.

# Providers

Datasource LWRP

```ruby
wildfly_datasource 'example' do
  jndiname 'java:jboss/datasource/example'
  drivername 'some-jdbc-driver'
  connectionurl 'jdbc:some://127.0.0.1/example'
  username 'db_username'
  password 'db_password'
  sensitive false  
end
```

Deploy LWRP

Allows you to deploy JARs and WARs via chef

Example 1 (from a url)
```ruby
wildfly_deploy 'jboss.jdbc-driver.sqljdbc4_jar' do
      url 'http://artifacts.company.com/artifacts/mssql-java-driver/sqljdbc4.jar'
end
```

Example 2 (from disk)
```ruby
wildfly_deploy 'jboss.jdbc-driver.sqljdbc4_jar' do
      path '/opt/resources/sqljdb4.jar'
end
```

Example 3 with automated update (requires a common runtime_name and version specific name)
```ruby
wildfly_deploy 'my-app-1.0.war' do
      url 'http://artifacts.company.com/artifacts/my-app.1.0.war'
      runtime_name 'my-app.war'
end
```

Example 4 undeploy (use :disable to keep the contents, and :enable to re-deploy previously kept contents)
```ruby
wildfly_deploy 'jboss.jdbc-driver.sqljdbc4_jar' do
      action :remove
end
```

Attribute LWRP

Allows you to set an attribute in the server config

To change the max-post-size parameter
```xml
            <server name="default-server">
			       <http-listener name="default" socket-binding="http" max-post-size="20971520"/>
				   <host name="default-host" alias="localhost">

```

```ruby
wildfly_attribute 'max-post-size' do
   path '/subsystem=undertow/server=default-server/http-listener=default'
   parameter 'max-post-size'
   value '20971520L'
   notifies :restart, 'service[wildfly]'
end
```

If the attribute restart is set to false, the wildfly will never restart

```ruby
wildfly_attribute 'max-post-size' do
   path '/subsystem=undertow/server=default-server/http-listener=default'
   parameter 'max-post-size'
   value '20971520L'
   restart false
end
```

Property LWRP

Allows you to set or delete system properties in the server config. (Supported Actions: :set, :delete)

```ruby
wildfly_property 'Database URL' do
   property 'JdbcUrl'
   value 'jdbc:mysql://1.2.3.4:3306/testdb'
   action :set
   notifies :restart, 'service[wildfly]', :delayed
end
```

## ChefSpec Matchers

This cookbook includes custom [ChefSpec](https://github.com/sethvargo/chefspec) matchers you can use to test 
your own cookbooks.

Example Matcher Usage

```ruby
expect(chef_run).to create_wildfly_datasource('example').with(
  jndiname 'java:jboss/datasource/example'
  drivername 'some-jdbc-driver'
  connectionurl 'jdbc:some://127.0.0.1/example'
  username 'db_username'
  password 'db_password'
  sensitive: true
)
```
      
Cookbook Matchers

* set_wildfly_attribute(resource_name)
* create_wildfly_datasource(resource_name)
* delete_wildfly_datasource(resource_name)
* install_wildfly_deploy(resource_name)
* remove_wildfly_deploy(resource_name)
* enable_wildfly_deploy(resource_name)
* disable_wildfly_deploy(resource_name)
* create_wildfly_logcategory(resource_name)
* delete_wildfly_logcategory(resource_name)
* create_wildfly_loghandler(resource_name)
* delete_wildfly_loghandler(resource_name)
* set_wildfly_property(resource_name)
* delete_wildfly_property(resource_name)

# Authors

Author:: Brian Dwyer - Intelligent Digital Services

# Contributors
Contributor:: Hugo Trippaers

Contributor:: Ian Southam

Contributor:: Lydia Joslin
