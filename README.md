# Knife sharp plugin

knife-sharp adds several handy features to knife, adapted to our workflow @ Fotolia.
Features:
* align : sync data bags, roles and cookbook versions between a local git branches and a chef server
* backup : dump environments, roles and data bags to local json files
* server : switch between chef servers using multiple knife.rb config files

# Tell me more

When you want an environment to reflect a given branch you have to check by hand (or using our consistency plugin), and some mistakes can be made. This plugin aims to help to push the right version into an environment.

It also allows to adopt a review workflow for main chef components :
* Track data bags, roles (as JSON files) and cookbooks in your Chef git repository
* Push each modification of any to peer-review
* Once merged, upload every change with knife sharp align

# Show me !

## Align

<pre>
$ git branch
...
master
* syslog_double
...
$ knife environment show sandboxnico
chef_type:            environment
cookbook_versions:
...
  syslog:  0.0.16
...
$ knife sharp align syslog_double sandboxnico

Will change in environment sandboxnico :
* syslog gets version 0.0.17
Upload and set version into environment sandboxnico ? Y/N
Y
Successfull upload for syslog
Aligning 1 cookbooks
Aligning data bags
* infrastructure/mail data bag item is not up-to-date
Update infrastructure/mail data bag item on server ? Y/N/(A)ll/(Q)uit [N] n
* Skipping infrastructure/mail data bag item
Aligning roles
* Dev_Server role is not up-to-date (run list)
Update Dev_Server role on server ? Y/N/(A)ll/(Q)uit [N] n
* Skipping Dev_Server role
</pre>

Then we can check environment :

<pre>
$ knife environment show sandboxnico
chef_type:            environment
cookbook_versions:
...
  syslog:  0.0.17
...
$ knife sharp align syslog_double sandboxnico
Nothing to do : sandboxnico has same versions as syslog_double
</pre>

It will upload the cookbooks (to ensure they meet the one on the branch you're working on) and will set the version to the required number.

## Backup

Making a backup before a large change can be a lifesaver. Knife sharp can do it for you, easily
<pre>
$ knife sharp backup
Backing up roles
Backing up environments
Backing up databags
$
</pre>

All these items get stored in the place defined in your config file.

## Server

<pre>
$ knife sharp server
Available servers:
   prod (/home/jamiez/.chef/knife-prod.rb)
>> dev (/home/jamiez/.chef/knife-dev.rb)

$ knife sharp server prod
The knife configuration has been updated to use prod.

$ knife sharp server
Available servers:
>> prod (/home/jamiez/.chef/knife-prod.rb)
   dev (/home/jamiez/.chef/knife-dev.rb)
</pre>

## Rollback

Sometimes you need to be able to rollback a change quickly, because failure happens. So knife sharp now creates rollback points of environment constraints.

A picture says a thousand words :

<pre>
$ knife sharp rollback --list
Available rollback points :
  * 1370414322 (Wed Jun 05 08:38:42 +0200 2013)
  * 1370418655 (Wed Jun 05 09:50:55 +0200 2013)
  * 1370419966 (Wed Jun 05 10:12:46 +0200 2013)
  * 1370421569 (Wed Jun 05 10:39:29 +0200 2013)
$ knife sharp rollback --show 1370421569
Rollback point has the following informations :
  environment : production
  cookbooks versions :
   * tests => = 0.0.2
   * varnish => = 0.0.10
$ knife sharp rollback --to 1370421569
Rollback point has the following informations :
  environment : production
  cookbooks versions :
   * varnish => = 0.0.10
   * tests => = 0.0.2
Continue rollback ? Y/(N) [N] y
Setting varnish to version = 0.0.10
Setting tests to version = 0.0.2
</pre>

The activation of this rollback feature and its storage dir can be configured in your sharp-config.yml file.

# Configuration

Dependencies :
* grit

The plugin will search in 2 places for its config file :
* "/etc/sharp-config.yml"
* "~/.chef/sharp-config.yml"

An example config file is provided in ext/.

A working knife setup is also required (cookbook/role/data bag paths depending on the desired features)

## Cookbooks path & git
If your cookbook_path is not the root of your git directory then the grit gem will produce an error. This can be circumvented by adding the following directive in your config file :

<pre>
global:
  git_cookbook_path: "/home/nico/sysadmin/chef/"
</pre>

As we version more than the cookbooks in the repo.

## Logging
It's good to have things logged. The plugin can do it for you. Add this to your config file
<pre>
logging:
  enabled: true
  destination: "~/.chef/sharp.log"
</pre>

It will log uploads, bumps and databags to the standard logger format.

# ZSH completion

want a completion on changing servers ?

<pre>
alias kss="knife sharp server"

function knife_servers { reply=($(ls .chef/knife-*.rb | sed -r 's/.*knife-([a-zA-Z0-9]+)\.rb/\1/' )); }
compctl -K knife_servers kss
</pre>

# Credits

The damn good knife spork plugin from the etsy folks : https://github.com/jonlives/knife-spork

Idea for knife sharp server comes from https://github.com/greenandsecure/knife-block

License
=======
3 clauses BSD

Authors
======
* Nicolas Szalay | https://github.com/rottenbytes
* Jonathan Amiez | https://github.com/josqu4red
