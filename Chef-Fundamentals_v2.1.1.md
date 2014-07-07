# Workstation Setup

- Install Chef - http://www.getchef.com/chef/install
- Get Chef - http://www.getchef.com

```$ curl -L http://www.getchef.com/chef/install.sh | sudo bash```

```$ cd chef-repo```

```$ ls -al```

```$ ls .chef``` 

```chef-repo/.chef/knife.rb```

```ruby
current_dir = File.dirname(__FILE__)
log_level              :info
log_location           STDOUT
node_name              "USERNAME"
client_key             "#{current_dir}/USERNAME.pem"
validation_client_name "ORGNAME-validator"
validation_key         "#{current_dir}/ORGNAME-validator.pem"
chef_server_url     "https://api.opscode.com/organizations/ORGNAME"
cache_type             'BasicFile'
cache_options( :path =>"#{ENV['HOME']}/.chef/checksums" )
cookbook_path          ["#{current_dir}/../cookbooks"]
```

```$ knife --version```

```$ knife client list```

```$ knife help list```

<!-- Bonus Exercises
Exercise #1
Situation:  
You want to keep your personal machines separate from your training environment.
Tasks:
• Create a second organization in your hosted Chef account called 
“<username>-home”. 
• Create a new Chef repo directory named “chef-repo-personal” and set it up to connect to your new organization.
• Create an “editor-test” client in your new personal organization.  Run “knife client list”.  Now change back into your training organization’s repo and run “knife client list” again.  What’s different?
• View your different organizations at http://manage.opscode.com 
• Don’t forget to change back into your training repo before we continue.
 -->

# Organization Setup
- Click the "Administration" tab, 
- Select the appropriate Organization
- Click "Invite User" from the left menu
- Enter your classmate's 'Chef Username' and click Invite

<image>

- Click the notification, select the Organization and click 'Accept'

<image>

- Select your classmate's organization from the drop down list and peruse their org

<image>

- Now either 'Leave Organization' you've been invited into, or remove your classmate from your organization

<image>
 
 
# Node Setup

```$ knife bootstrap <EXTERNAL_ADDRESS> --sudo -x chef -P chef -N "node1"```

```$ ssh chef@IPADDRESS```

```chef@node1:~$ ls /etc/chef```

```chef@node1:~$ which chef-client```

```chef@node1:~$ cat /etc/chef/client.rb```

```chef@node1:~$ sudo vi /etc/chef/client.rb```

```Set log_level to :info```

- View Node on Chef Server 

<image>

<!-- Bonus Exercises
Exercise #1
Situation:  
A junior admin accidentally deleted client.pem.  Your job is to repair the damage.
Tasks:
• Delete /etc/chef/client.pem from your target node and run 
“sudo chef-client”.  What happens?
• How can you fix the target host so it can communicate with the server again?  (Hint:  Every node also has a client)

Exercise #2
Situation:  
Your target node is not authenticating to the server and you’re not sure why.
Tasks:
• Set the time on your target to midnight with this command:  
date +%T -s "00:00:00"
• Run “sudo chef-client”.  What happens?  Why?
• Correct the time with this command:  ntpdate pool.ntp.org

Exercise #1 solution:
* Run “knife node delete target1” and “knife client delete target1” on the workstation.
* Run “sudo chef-client” on the server
 -->
 
# Chef Resources and Recipes

```$ knife cookbook create apache```

```$ ls -la cookbooks/apache```



```ruby
cookbooks/apache/recipes/default.rb

package "httpd" do
  action :install
end

service "httpd" do
  action [ :enable, :start ]
end

cookbook_file "/var/www/html/index.html" do
  source "index.html"
  mode "0644"
end
```


```html
cookbooks/apache/files/default/index.html

<html>
<body>
  <h1>Hello, world!</h1>
</body>
</html>
```

```$ knife cookbook upload apache```

```$ knife node run_list add node1 "recipe[apache]"```

```chef@node1:~$ sudo chef-client```

<!-- Bonus Exercises
Exercise #1
Situation:  
Your boss wants all webservers running CentOS to  have a homepage that says “Hello CentOS World”
Tasks:
• Set up a custom index.html file that will *only* be used on  servers running Ubuntu.  (Substitute with ubuntu if you like!)

Exercise #2
Situation:  
Your web developer is complaining that his web application is not showing the latest content, and she wants the web server to be restarted when the content is updated.
Tasks:
• Make a small change to the index.html file you created in Exercise #1
• In your default recipe, use the “notifies” notification to tell the apache2 service to restart whenever index.html is updated.  Don’t forget to re-upload your apache cookbook before running chef-client on the target.

Exercise #1 solution:
1.  Create ~/chef-repo/cookbooks/apache/files/centos
2.  Put your new “Hello CentOS World” index.html file in that directory
3.  Run “knife cookbook upload apache” on the workstation
4.  Run “sudo chef-client” on the target

Exercise #2 solution:
1.  Edit your default recipe and add this line to the cookbook_file resource:
  notifies :restart, "service[apache2]"
2.  Make a small change to your default/index.html (or ubuntu/index.html) file
3.  Run “knife cookbook upload apache” on the workstation
4.  Run “sudo chef-client” on the target

NOTE:  Restarting apache is actually not necessary here, but it's a good way to introduce the notifies notification.
 -->


# Introducing the Node Object

```$ knife node list```

```$ knife client list```

```$ knife node show node1```

```chef@node1:~$ sudo ohai | less```

```$ knife node show node1 -l```

```$ knife node show node1 -Fj```

```$ knife node show node1 -a fqdn```

```$ knife search node "*:*" -a fqdn```


 
# Node Attributes

```ruby
cookbooks/apache/attributes/default.rb

default["apache"]["indexfile"] = "index1.html"
```

```html
cookbooks/apache/files/default/index1.html

<html>
 <body>
  <h1>Hello, world!</h1>
   <h2>This is index1.html</h2>
    <p>We configured this in the attributes file</p>
 </body>
</html>
```

```ruby
cookbooks/apache/recipes/default.rb

cookbook_file "/var/www/html/index.html" do
  source node["apache"]["indexfile"]
  mode "0644"
end
```

```$ knife cookbook upload apache```

```chef@node1:~$ sudo chef-client```

```ruby
cookbooks/apache/recipes/default.rb

node.default["apache"]["indexfile"] = "index2.html"
cookbook_file "/var/www/html/index.html" do
  source node["apache"]["indexfile"]
  mode "0644"
end
```

```html
cookbooks/apache/files/default/index2.html

<html>
 <body>
  <h1>Hello, world!</h1>
   <h2>This is index2.html</h2>
    <p>We configured this in the recipe</p>
 </body>
</html>
```


```$ knife cookbook upload apache```

```chef@node1:~$ sudo chef-client```
 

# Attributes, Templates, and Cookbook Dependencies

*Use knife to create a cookbook called 'motd' (command hidden)*
<!-- $ knife cookbook create motd -->


```ruby
cookbooks/motd/attributes/default.rb

default["motd"]["company"] = "Chef"
```

*Add template resource to the motd cookbook's default recipe (cookbooks/motd/recipes/default.rb) for /etc/motd based on the source 'motd.erb'. (command hidden)*
<!-- template "/etc/motd" do
   source "motd.erb"
   mode "0644"
 end -->



```ruby
cookbooks/motd/templates/default/motd.erb

This server is property of <%= node["motd"]["company"] %>
 <% if node["pci"]["in_scope"] -%>
 This server is in-scope for PCI compliance
 <% end -%>
```

*Use knife upload the 'motd' cookbook (command hidden)*
<!-- $ knife cookbook upload motd -->


*Use knife to create a cookbook called 'pci' (command hidden)*

```ruby
cookbooks/pci/attributes/default.rb

default["pci"]["in_scope"] = true
```

*Use knife upload the 'pci' cookbook (command hidden)*
<!-- $ knife cookbook upload pci -->


 *Use knife add 'recipe[motd]' to node1's run list (command hidden)*
<!-- $ knife node run_list add node1 "recipe[motd]" -->


```$ knife node show node1```

```chef@node1:~$ sudo chef-client```

```ruby
cookbooks/motd/metadata.rb

maintainer       "YOUR_COMPANY_NAME"
maintainer_email "YOUR_EMAIL"
license          "All rights reserved"
description      "Installs/Configures motd"
long_description IO.read(File.join(File.dirname(__FILE__), ‘README.md‘))
version          "0.1.0"
depends "pci"
```

*Use knife upload the 'motd' cookbook (command hidden)*
<!-- $ knife cookbook upload motd -->

*Rerun 'chef-client' on node1 (command hidden)*
<!-- opscode@node1:~$ sudo chef-client -->

```ruby
cookbooks/pci/attributes/default.rb

default["pci"]["in_scope"] = false
```

*Use knife upload the 'pci' cookbook (command hidden)*
<!-- $ knife cookbook upload pci -->

*Rerun 'chef-client' on node1 (command hidden)*
<!-- opscode@node1:~$ sudo chef-client -->

```chef@node1:~$ cat /etc/motd```

```$ knife node show node1 -a pci```

<!-- Bonus Exercises
Exercise #1
Situation:  
You need to list the IP addresses of only linux nodes (CentOS or Ubuntu)
Tasks:
• Use the “knife search” command to create your list.  (HINT:  You can use the -l flag to get a list of all the attributes that are available to you.)

Exercise #2
Situation:  
The pretty MOTD banner is gone!
Tasks:
• Restore the /etc/motd banner from a backup.  Where might the backup of this file  be located?  HINT:  Look in the chef-client output.  Run chef-client again. What happens?

Exercise #3
Situation:  
The client wants the hostname of the machine to be automatically included on their homepage.
Tasks:
• Edit your motd.erb template, replacing “This server” with the node’s hostname using a node attribute and embedded Ruby.

Exercise #1 solution:
knife search node "os:linux" -a ipaddress

Exercise #2 solution:
knife search node “recipes:apache”

Exercise #3 solution:
1. Edit the motd.erb file and replace all instances of “This server” with <%= node["hostname"] -%>
2. Run “knife cookbook upload motd” on your workstation
3. Run “sudo chef-client” on the target node
 -->

 
# Template Variables, Notifications, and Controlling Idempotency

```ruby
cookbooks/apache/metadata.rb

maintainer       "YOUR_COMPANY_NAME"
maintainer_email "YOUR_EMAIL"
license          "All rights reserved"
description      "Installs/Configures apache"
long_description IO.read(File.join(File.dirname(__FILE__), 'README.md'))
version          "0.2.0"
```

```ruby
cookbooks/apache/attributes/default.rb

default["apache"]["sites"]["clowns"] = { "port" => 80 }
default["apache"]["sites"]["bears"] = { "port" => 81 }
```

```ruby
cookbooks/apache/recipes/default.rb
(See https://gist.github.com/6781185)

package "httpd" do
  action :install
end
 
service "httpd" do
  action [ :enable, :start ]
end
 
execute "mv /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.conf.disabled" do
  only_if do
    File.exist?("/etc/httpd/conf.d/welcome.conf")
  end
  notifies :restart, "service[httpd]"
end
 
node["apache"]["sites"].each do |site_name, site_data|
  document_root = "/srv/apache/#{site_name}"
 
  template "/etc/httpd/conf.d/#{site_name}.conf" do
   source "custom.erb"
   mode "0644"
   variables(:document_root => document_root,:port => site_data["port"])
   notifies :restart, "service[httpd]"
  end
 
  directory document_root do
   mode "0755"
   recursive true
  end
 
  template "#{document_root}/index.html" do
   source "index.html.erb"
   mode "0644"
   variables(:site_name => site_name, :port => site_data["port"])
  end
end
```

```ruby
cookbooks/apache/templates/default/custom.erb
(See https://gist.github.com/8955103)

<% if @port != 80 -%>
  Listen <%= @port %>
<% end -%>

<VirtualHost *:<%= @port %>>
  ServerAdmin webmaster@localhost

  DocumentRoot <%= @document_root %>
  <Directory />
  Options FollowSymLinks
  AllowOverride None
  </Directory>
  <Directory <%= @document_root %>
  Options Indexes FollowSymLinks MultiViews
  AllowOverride None
  Order allow,deny
  allow from all
  </Directory>
</VirtualHost>
```

```html
cookbooks/apache/templates/default/index.html.erb
(See https://gist.github.com/2866421)

<html>
  <body>
    <h1>Welcome to <%= node["motd"]["company"] %></h1>
    <h2>We love <%= @site_name %></h2>
    <%= node["ipaddress"] %>:<%= @port %>
  </body>
</html>
```

*Use knife upload the 'apache' cookbook (command hidden)*
<!-- $ knife cookbook upload apache -->
*Rerun 'chef-client' on node1 (command hidden)*
<!-- opscode@node1:~$ sudo chef-client -->

<!-- - Troubleshoot the failure

```ruby
cookbooks/apache/templates/default/custom.erb

  <Directory <%= @document_root %>>
```

```opscode@node1:~$ sudo chef-client```

*cookbooks/apache/recipes/default.rb*
- Move 'service' resource to end of recipe

```opscode@node1:~$ sudo chef-client``` -->


<!-- Bonus Exercises
Exercise #1
Situation:  
Someone borked your Apache cookbook, and now Apache won’t start properly.  
Tasks:
• Replace your “custom.erb” with the contents of this one:
 https://gist.github.com/scarolan/6091028
• Run “knife cookbook upload apache” on your workstation
• Stop apache on the target host with “sudo service apache2 stop”
• Attempt to run chef-client again.  It will probably fail.  Why did it fail?  Fix it without manually changing any configurations on the target.  You may only use chef-client to get Apache running again.

Exercise #2
Situation:  
The marketing people are convinced that ponies are the new hotness.
Tasks:
• Create a  new site running on port 83 for ponies.  Don’t use port 82, we are going to use that for  something else later in the training.

Exercise #3 - Advanced
Situation:  
The boss wants more pictures on our websites.
Tasks:
• Figure out how to add an image to each of your websites’ home pages, using default attributes.  You can search Google images and use a remotely hosted file if you wish.   
HINT:  HTML formatting for images looks like this:  
<img src=”http://www.example.com/path/to/image.jpg”>
• Complete solution is here, don’t peek unless you are completely stuck!  https://gist.github.com/scarolan/6091430
• NOTE:  The solution file is for Ubuntu systems but the method to get the images working is *identical*.

Exercise #1 solution:
There are two parts to this problem.  The first is the typo we introduced on line 13 of custom.erb; there is a missing > at the end of the line.  But even if the student fixes that, Apache still won’t start.  The reason is because we have the “service” resource before the “template” resource in the default recipe.  This is a learning opportunity - show the student how order matters, and how Chef will stop if it encounters any errors.  It’s also a good opportunity to learn the difference between Chef-related errors, and OS-related errors that caused the chef-client to stop running.

Exercise #2 solution:
Simply edit the ./cookbooks/apache/attributes/default.rb, adding a ponies line underneath the clowns and bears lines.  Save and re-upload the cookbook.

Exercise #3  solution:
I posted it here:  https://gist.github.com/scarolan/6091430
Basically you just add an “image” attribute for each site inside the default attributes file. -->

 
# Search

```$ knife search node "*:*"```

```$ knife search node "ipaddress:10.*"```

```$ knife search node "*:*" -a ipaddress```

```$ knife search node "ipaddress:10.*" -a ipaddress```

```$ knife search node "ipaddress:10* AND platform:centos"```

```$ knife search node "ipaddress:[10.0.* TO 10.2.*]"```

```ruby
cookbooks/apache/recipes/ip-logger.rb

search("node","platform:centos").each do |server|
  log "The CentOS servers in your organization have the following FQDN/IP Addresses:- #{server["fqdn"]}/#{server["ipaddress"]}"
end
```

*Use knife upload the 'apache' cookbook (command hidden)*
<!-- $ knife cookbook upload apache -->

*Add the recipe ' apache::ip-logger' to node1's run list (command hidden)*
<!-- $ knife node run_list add node1 'recipe[apache::ip-logger]' -->

*Rerun 'chef-client' on node1 (command hidden)*
<!-- opscode@node1:~$ sudo chef-client -->

*Remove the recipe ' apache::ip-logger' from node1's run list (command hidden)*
<!-- $ knife node run_list remove node1 'recipe[apache::ip-logger]' -->
 


# Recipe Inclusion, Data Bags, and Search
```$ mkdir –p data_bags/users```

```$ knife data_bag create users```

```ruby
data_bags/users/bobo.json

{
  "id": "bobo",
  "comment": "Bobo T. Clown",
  "uid": 2000,
  "gid": 0,
  "home": "/home/bobo",
  "shell": "/bin/bash"
}
```

```$ knife data_bag from file users bobo.json```

*Create another user in the users data bag called 'frank' (command hidden)*
<!-- data_bags/users/frank.json -->

```ruby
{
  "id": "frank",
  "comment": "Frank Belson",
  "uid": 2001,
  "gid": 0,
  "home": "/home/frank",
  "shell": "/bin/bash"
}
```

*Use knife to upload frank's data_bag item(command hidden)*
<!-- $ knife data_bag from file users frank.json -->

```$ knife search users "*:*"```

```$ knife search users "id:bobo" -a shell```

*Create a data_bag called 'groups' (2 commands hidden)*
<!-- $ mkdir data_bags/groups
$ knife data_bag create groups -->

```ruby
data_bags/groups/clowns.json

{
  "id": "clowns",
  "gid": 3000,
  "members": [ "bobo", "frank" ]
}
```

*Use knife to upload the 'clowns' data_bag item (command hidden)*
<!-- $ knife data_bag from file groups clowns.json -->

```$ knife search groups "*:*"```

*Create a cookbook called 'users' (command hidden)*
<!-- $ knife cookbook create users -->

- Edit the 'user' cookbook's default recipe and add the following 

```ruby
cookbooks/users/recipes/default.rb

search(:users, "*:*").each do |user_data|
  user user_data["id"] do
    comment user_data["comment"]
    uid user_data["uid"]
    gid user_data["gid"]
    home user_data["home"]
    shell user_data["shell"]
  end
end
include_recipe "users::groups"

cookbooks/users/recipes/groups.rb
search(:groups, "*:*").each do |group_data|
  group group_data["id"] do
    gid group_data["gid"]
    members group_data["members"]
  end
end
```

*Upload the 'users' cookbook (command hidden)*
<!-- $ knife cookbook upload users -->

*Use knife to add the 'users' cookbook's default receipt to node1's run list (command hidden)*
<!-- knife node run_list add node1 'recipe[users]' -->

*Rerun 'chef-client' on node1 (command hidden)*
<!-- opscode@node1:~$ sudo chef-client -->

```chef@node1:~$ cat /etc/passwd```

```chef@node1:~$ cat /etc/group```

<!-- Bonus Exercises
Exercise #1
Situation:  
Frank and Bobo have user accounts but no home directories.  (RHEL/CentOS users can skip this exercise.)
Tasks:
• Use your knowledge and the Chef documentation to figure out how to have user home directories created automatically.

Exercise #2
Situation:  
A new junior admin named Zippy started work this morning. You need to give him a restricted account.
Tasks:
• Create a new user account for Zippy, and set his default shell to “/bin/rbash” instead of “/bin/bash”.  Set Zippy’s uid to 2002, and his gid to 0.  Don’t forget to add him to the clowns group in your group data bag.


Exercise #1 solution:
This exercise is not necessary for RHEL/CentOS users because home directories get added by default, even without manage_home.

This is a tricky one because even if the student gets so far as finding the “manage_home” setting in the documentation, it still won’t work for *existing* users due to this long-standing bug.  Take this as an opportunity to explain how to submit bugs, and that it’s almost always better to rebuild from scratch than to try and edit things that are already in place.

http://tickets.opscode.com/browse/CHEF-2409

Solution #1:
1. Add this to the user resource in our users cookbook:
supports :manage_home => true
2. Delete Frank and Bobo with the “userdel” command, re-run chef-client

Solution #2:
Use a new “directory” resource inside the each loop to ensure that their home directories are created properly.

Exercise #2 solution:
Copy bobo.json to zippy.json
Edit zippy.json, changing his uid to 2002, and his shell to “/bin/rbash”
Run “knife data bag from file zippy.json
Edit clowns.json and add zippy to the list of clowns in the group
Run “knife data bag from file clowns.json”
Run “sudo chef-client” on the target

NOTE:  If the student does exercise #2 *before* the encrypted data bags section, make sure they configure a password for Zippy there as well.
 --> 


# Roles

```ruby
roles/webserver.rb

name "webserver"
description "Web Server"
run_list "recipe[apache]"
default_attributes({
  "apache" => {
    "sites" => {
      "admin" => {
        "port" => 8000
      }
    }
  }
})
```

```$ knife role from file webserver.rb```

```$ knife role show webserver```

*Use knife to search for roles that have the apache cookbook's default recipe in its run_list (command hidden)*
<!-- $ knife search role "run_list:recipe\[apache\]" -->

- Replace recipe[apache] with role[webserver] in run list
<image>

```chef@node1:~$ sudo chef-client```

```$ knife search node "role:webserver" -a apache.sites```

```ruby
roles/webserver.rb

name "webserver"
description "Web Server"
run_list "recipe[apache]"
default_attributes({
  "apache" => {
    "sites" => {
      "admin" => {
        "port" => 8000
      },
      "bears" => {
        "port" => 8081
      }
    }
  }
})
```

*Use knife to upload this webserver role & rerun chef-client (commands hidden)*
<!-- $ knife role from file webserver.rb -->

```opscode@node1:~$ sudo chef-client```

*Use knife to Display the 'apache.sites' attribute on all nodes with webserver role (command hidden)*
<!-- $ knife search node "role:webserver" -a apache.sites -->

Edit the 'base' role (command hidden)
<!-- roles/base.rb -->

```ruby
name "base"
description "Base Server Role"
run_list "recipe[motd]", "recipe[users]"
```

*Upload the 'base' role to Chef server (command hidden)*
<!-- $ knife role from file base.rb -->

*Edit the 'webserver' role (command hidden)*
<!-- roles/webserver.rb -->

```ruby
name "webserver"
description "Web Server"
run_list "role[base]", "recipe[apache]"
default_attributes({
  "apache" => {
    "sites" => {
      "admin" => {
        "port" => 8000
      },
      "bears" => {
        "port" => 8081
      }
    }
  }
})
```

*Upload the 'webserver' role to Chef server (command hidden)*
<!-- $ knife role from file webserver.rb -->

*Rerun 'chef-client' on node1 (command hidden)*
<!-- opscode@node1:~$ sudo chef-client -->


<!-- Bonus Exercises
Exercise #1
Situation:  
You need a dedicated NTP server inside the PCI environment to sync the clocks on all your hosts.

Tasks:
• Create a new role called “ntp_server”.  It’s run list should be empty for now, we’ll use it in a later exercise. 

Exercise #1 solution:
Just create an ntpserver.rb file inside of ./roles, and edit appropriately.  Run “knife role from file ntpserver.rb” when done.
 -->
 
# Environments
```$ knife cookbook show apache```

```$ knife environment list```

```$ mkdir environments```

```ruby
environments/dev.rb

name "dev"
description "For developers!"
cookbook "apache", "= 0.2.0"
```

```$ knife environment from file dev.rb```

```$ knife environment show dev```

- Use the UI to change your node’s environment to "dev"
<image>


*Rerun 'chef-client' on node1 (command hidden)*
<!-- opscode@node1:~$ sudo chef-client -->

```ruby
environments/production.rb

name "production"
description "For Prods!"
cookbook "apache", "= 0.1.0"
override_attributes({
 "pci" => {
   "in_scope" => true
 }
})
```

*Use knife to upload this production environment (command hidden)*
<!-- $ knife environment from file production.rb -->

*Use the UI to change your node’s environment to "production" (Screenshot hidden)*
<!-- <image hidden> -->
 
*Rerun 'chef-client' on node1 (command hidden)*
<!-- opscode@node1:~$ sudo chef-client -->

<!-- Bonus Exercises
Exercise #1
Situation:  
Mordac from the security team has insisted that there be a completely separate PCI dev environment.

Tasks:
• Create a new environment called “dev_pci”.  It should be identical to the production environment, but with Apache version 0.2.0 instead of 0.1.0.  Try putting your target node in this environment to see what happens. 

Exercise #1 solution:
Just create an ntpserver.rb file inside of ./roles, and edit appropriately.  Run “knife role from file ntpserver.rb” when done. -->

 
#Using Community Cookbooks
```$ knife cookbook site search chef-client```

```$ knife cookbook site show chef-client```

```$ knife cookbook site download chef-client```

```$ tar -zxvf chef-client*.tar.gz -C cookbooks/```

```ruby
cookbooks/chef-client/recipes/delete_validation.rb

unless chef_server?
  file Chef::Config[:validation_key] do
    action :delete
    backup false
    only_if { ::File.exists?(Chef::Config[:client_key]) }
  end
end
```

```ruby
roles/base.rb

name "base"
description "Base Server Role"
run_list "recipe[chef-client::delete_validation]", "recipe[motd]", "recipe[users]"
```

```ruby
cookbooks/chef-client/recipes/default.rb

include_recipe "chef-client::service"

cookbooks/chef-client/recipes/service.rb
supported_init_styles = [
  'arch',
  'bluepill',
  'bsd',
  'daemontools',
  'init',
  'launchd',
  'runit',
  'smf',
  'upstart',
  'winsw'
]
init_style = node["chef_client"]["init_style"]

# Services moved to recipes
if supported_init_styles.include? init_style
  include_recipe "chef-client::#{init_style}_service"
else
  log "Could not determine service init style, manual intervention required to start up the chef-client service."
end
```

*Use knife to upload the 'chef-client' cookbook (command hidden)*
<!-- $ knife cookbook upload chef-client -->

*Use knife to download the 'cron' cookbook (command hidden)*
<!-- $ knife cookbook site download cron -->

*untar the 'cron' cookbook into the cookbooks directory (command hidden)*
<!-- $ tar -zxvf cron*.tar.gz -C cookbooks/ -->

*Use knife to upload the 'cron' cookbook (command hidden)*
<!-- $ knife cookbook upload cron -->

*Use knife to upload the 'chef-client' cookbook (command hidden)*
<!-- $ knife cookbook upload chef-client -->

*Use knife to download the 'logrotate' cookbook (command hidden)*
<!-- $ knife cookbook site download logrotate -->

*untar the 'logrotate' cookbook into the cookbooks directory (command hidden)*
<!-- $ tar -zxvf logrotate*.tar.gz -C cookbooks/ -->

*Use knife to upload the 'logrotate' cookbook (command hidden)*
<!-- $ knife cookbook upload chef-client -->

*Use knife to upload the 'chef-client' cookbook (command hidden)*
<!-- $ knife cookbook upload logrotate -->

*Edit the 'base' role (command hidden)*
<!-- roles/base.rb -->

```ruby
name "base"
description "Base Server Role"
run_list "recipe[chef-client::delete_validation]", "recipe[chef-client]", "recipe[motd]", "recipe[users]"
```

*Upload the 'base' role to Chef server (command hidden)*
<!-- $ knife role from file base.rb -->

*Rerun 'chef-client' on node1 (command hidden)*
<!-- opscode@node1:~$ sudo chef-client -->

*Check the 'chef-cllient' service is running (command hidden)*
<!-- opscode@node1$ ps awux | grep chef-client -->

*Use knife to download the 'ntp' cookbook (command hidden)*
<!-- $ knife cookbook site download ntp -->

*untar the 'ntp' cookbook into the cookbooks directory (command hidden)*
<!-- $ tar -zxvf ntp*.tar.gz -C cookbooks/ -->

*Use knife to upload the 'ntp' cookbook (command hidden)*
<!-- $ knife cookbook upload ntp -->

*Edit the 'base' role (command hidden)*
<!-- roles/base.rb -->

```ruby
name "base"
description "Base Server Role"
run_list "recipe[chef-client::delete_validation]", "recipe[chef-client]", "recipe[ntp]", "recipe[motd]", "recipe[users]"
```

*Upload the 'base' role to Chef server (command hidden)*
<!-- $ knife role from file base.rb -->

<!-- Bonus Exercises
Exercise #1
Situation:  
You need to sync all your hosts in with specific NTP servers.
Tasks:
• Without editing any cookbook code or node objects, set the default NTP servers to these North America NTP Pool hosts:
  server 0.north-america.pool.ntp.org
  server 1.north-america.pool.ntp.org
  server 2.north-america.pool.ntp.org
  server 3.north-america.pool.ntp.org

Exercise #2
Situation:  
Mordac is back and says you have to have an internal NTP server for PCI hosts.
Tasks:
• Transform your target into an NTP server.  HINT:  Reading is fun!

Exercise #1 solution:
Drop this into your base.rb role file and update the base role:

default_attributes({
  'ntp' => {
    'servers' => [ "0.north-america.pool.ntp.org", "1.north-america.pool.ntp.org", "2.north-america.pool.ntp.org", "3.north-america.pool.ntp.org" ]
  }
})

Exercise #2 solution:  The answer to this one is also right there in the README.md for the NTP cookbook:

Then in an ntpserver.rb role that is applied to NTP servers (e.g., time.int.example.org):
 
    name "ntp_server"
    description "Role applied to the system that should be an NTP server."
    default_attributes(
      "ntp" => {
        "is_server" => "true",
        "servers" => ["0.pool.ntp.org", "1.pool.ntp.org"],
        "peers" => ["time0.int.example.org", "time1.int.example.org"],
        "restrictions" => ["10.0.0.0 mask 255.0.0.0 nomodify notrap"]
      }
    ) -->


 
# Just Enough Ruby for Chef

<!-- Bonus Exercises
Exercise #1
Situation:  
You need to learn more Ruby because you want to be a Chef ninja.
Tasks:
• Sign up for an account on codeacademy.com and start doing the exercises in the Ruby track.  http://www.codecademy.com/tracks/ruby
 -->


