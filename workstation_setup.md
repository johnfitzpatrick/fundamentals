# Workstation Setup

## Workstation Setup - Mac OS X / Linux

    $ curl -L http://www.opscode.com/chef/install.sh | sudo bash


### Notes

Alternatively, Chef may be programmatically installed via the install.sh script.  This is the method used during chef bootstraps.

This installs chef-client, knife, ohai and ruby along with any supporting ruby gems.  This is exactly the same as gets installed on a node during bootstrap.  Difference is the node does not use knife, while the workstation client does not use ohai - but each could be configured to perform either role!