# Policyfiles Training Lab
A training lab on how to use Chef Policyfiles

Policies are built by defining a Policyfile, which looks similar to a Chef Role combined with a Berksfile. 

When a Policy is ready for upload, a workstation command included with the ChefDK compiles the Policyfile into a ```Policyfile.lock``` file. This locked Policy, along with all of the cookbooks it references, are treated as a single unit by the Chef tooling. The bundle of ```Policyfile.lock``` and cookbooks are uploaded to the server simultaneously. 
## Why Policyfiles ?
Policyfiles provide cookbook dependency management and replaces roles and environments. This allows you to get exact, repeatable results !

Policies make your chef-client runs completely repeatable, because cookbooks referenced in a Policy are identified by a unique hash based on their contents. This means that once the lock file + cookbook bundle has been generated, the code underlying it will never change.

## Step 1: Create a base policyfile
The base policyfile will be used by all the nodes.

Run the following command in the directory ```C:\Users\chef\cookbooks\policyfiles>```. If the directory does not exist, create it:
```
$ code base.rb
```

In Visual Studio Code, create the following:

```
# base.rb - Describe how you want Chef to build your system.
#
# For more information on the Policyfile feature, visit
# https://github.com/opscode/chef-dk/blob/master/POLICYFILE_README.md

# A name that describes what the system you're building with Chef does.
name 'base'

# Where to find external cookbooks:
default_source :supermarket

# Specify a custom source for a cookbook:
cookbook 'chef-client', git: 'https://github.com/anthonygrees/chef-client.git'
cookbook 'audit_agr', git: 'https://github.com/anthonygrees/audit_agr.git'

# run_list: chef-client will run these recipes in the order specified.
run_list 'audit_agr', 'chef-client'

```

## Step 2: Generate the ```lock`` file
With our basic base.rb policyfile, we run ```chef install`` to fetch dependencies and generate a ```base.lock.json```.

So run the following command:
```
$ chef install base.rb
```
and you will see the following output
```
C:\Users\chef\cookbooks\policyfiles> chef install base.rb
Building policy base
Expanded run list: recipe[audit_agr], recipe[chef-client]
Caching Cookbooks...
Installing chef-client >= 0.0.0 from git
Installing audit_agr   >= 0.0.0 from git
Installing cron        6.2.1
Installing logrotate   2.2.0
Installing windows     5.2.3
Installing audit       7.3.0

Lockfile written to C:/Users/chef/cookbooks/policyfiles/base.lock.json
Policy revision id: f458a363e1ed148676a5ee5c9a558cb0dd3ba8581803de44de49ccd7b1d5e134
C:\Users\chef\cookbooks\policyfiles>
```

## Step 3: Let's take a look at the lock file
Let’s take a look at the base.lock.json we just created. We’ll go over each part individually:

### Revision ID
Each time we create or update the lock, chef will automatically generate a revision_id based on the content. These values are used to automatically version your policies, so that you can apply different revsions of a policy to different set of servers. We’ll see this in action a little later.
```
"revision_id": "5f750bf464100b487cd7c276c5d532341b79fbeb5e8accd29538ae972896992b"
```

### Name and Run List
The lock includes the name and run list we specified previously. The run list is normalized to the least ambiguous form.

In this example, you will see that it has selected the ```default``` recipe for both of the cookbooks.
```
"run_list": [
    "recipe[audit_agr::default]",
    "recipe[chef-client::default]"
```

### Cookbook Locks
For each cookbook we use, there is a corresponding entry in the ```cookbook_locks``` section. The exact data collected about each cookbook is dependent on the cookbook’s source. In this case, we have a cookbook sourced from GitHub which is a git repo. In the event we need to debug this cookbook later, ChefDK has collected information about the cookbook’s git revision. 

You will notice that it has included more cookbooks than we specified.  This is because the ```Audit``` and ```Chef_Client``` cookbooks have dependant cookbooks.  The policyfile tracks each and every one !

```
  "cookbook_locks": {
    "audit": {
      "version": "7.3.0",
      "identifier": "ec19259446ed7259f898861c30d1b106d649616f",
      "dotted_decimal_identifier": "66455743695875442.25324606895960273.194642923053423",
      "cache_key": "audit-7.3.0-supermarket.chef.io",
      "origin": "https://supermarket.chef.io:443/api/v1/cookbooks/audit/versions/7.3.0/download",
      "source_options": {
        "artifactserver": "https://supermarket.chef.io:443/api/v1/cookbooks/audit/versions/7.3.0/download",
        "version": "7.3.0"
      }
    },
    "audit_agr": {
      "version": "2.2.2",
      "identifier": "997012b6a9e2488cfead390e2fdf26821a3cf3cf",
      "dotted_decimal_identifier": "43188897113039432.39686516679520223.42340227806159",
      "cache_key": "audit_agr-de245088a67ed9a178e39ad33b9868d95c0a20df",
      "origin": "https://github.com/anthonygrees/audit_agr.git",
      "source_options": {
        "git": "https://github.com/anthonygrees/audit_agr.git",
        "revision": "de245088a67ed9a178e39ad33b9868d95c0a20df"
      }
    },
    "chef-client": {
      "version": "10.2.2",
      "identifier": "665de50495d89d717cd73c0cf68a15a847a621d8",
      "dotted_decimal_identifier": "28813685830310045.31943936235599498.23812500759000",
      "cache_key": "chef-client-65efbdbc8d8df7077378974a07455be06699708d",
      "origin": "https://github.com/anthonygrees/chef-client.git",
      "source_options": {
        "git": "https://github.com/anthonygrees/chef-client.git",
        "revision": "65efbdbc8d8df7077378974a07455be06699708d"
      }
    },
    "cron": {
      "version": "6.2.1",
      "identifier": "08676b5cc33658430ce04cecc16dd07200380e83",
      "dotted_decimal_identifier": "2365510629144152.18872980942405997.229188048522883",
      "cache_key": "cron-6.2.1-supermarket.chef.io",
      "origin": "https://supermarket.chef.io:443/api/v1/cookbooks/cron/versions/6.2.1/download",
      "source_options": {
        "artifactserver": "https://supermarket.chef.io:443/api/v1/cookbooks/cron/versions/6.2.1/download",
        "version": "6.2.1"
      }
    },
    "logrotate": {
      "version": "2.2.0",
      "identifier": "53e09234a4f73cc13f46d833d2e5075cafddfaa8",
      "dotted_decimal_identifier": "23609341620057916.54394244012692197.8094668946088",
      "cache_key": "logrotate-2.2.0-supermarket.chef.io",
      "origin": "https://supermarket.chef.io:443/api/v1/cookbooks/logrotate/versions/2.2.0/download",
      "source_options": {
        "artifactserver": "https://supermarket.chef.io:443/api/v1/cookbooks/logrotate/versions/2.2.0/download",
        "version": "2.2.0"
      }
    },
    "windows": {
      "version": "5.2.3",
      "identifier": "b9450a2483840dd3f3e9044b17eb8c5c2fec3178",
      "dotted_decimal_identifier": "52148780556059661.59659402210908139.154327568888184",
      "cache_key": "windows-5.2.3-supermarket.chef.io",
      "origin": "https://supermarket.chef.io:443/api/v1/cookbooks/windows/versions/5.2.3/download",
      "source_options": {
        "artifactserver": "https://supermarket.chef.io:443/api/v1/cookbooks/windows/versions/5.2.3/download",
        "version": "5.2.3"
      }
    }
  },
  ```
  


```
# Override the Chef Client cookbook with the following attributes
override['chef_client']['interval']    = '200'
override['chef_client']['splay']       = '30'

override['audit']['profiles']['linux-patch-baseline'] = { 'url': 'https://github.com/dev-sec/linux-patch-baseline/archive/0.4.0.zip' }
```
