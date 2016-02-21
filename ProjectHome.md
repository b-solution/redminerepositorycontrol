Adds fine grained permissions for repository access to redmine through Apache.


# How the permissions work #

Controls are based on Roles. The permissions browse\_repository and commit\_access are used to see if a role has permissions to perform and action, and the role permissions set in the administration page are treated as the default permissions for authorizing access.

However, if a role that is higher than user's given role in a project has an explicit permission set in the Repository Controls page, they will be denied access.

For instance, you have two users, Manager1 and Developer1, on a project called MyProject. Manager1 is assigned to the Manager role, and Developer1 is assigned to the Developer role, both roles have browse and commit access enabled. You also have a repository with a structure as such:

```
 - trunk/
 ----- Management_Files/
 ------------ Salaries/
 ----- Code/
```

By default, since both roles have browse and commit access, Developer1 and Manager1 can see all of the files in the repository. If we add a control that says the Developer role has no permissions to the path "/trunk/Management\_Files", now Developer1 will not be able to access /trunk/Management\_Files at all (read or write), but he will still be able to access /trunk, and /trunk/Code.

Say then we add a new user, Reporter1, to the project, and assign him the Reporter role. The reporter role, in our example, also has browse and commit access. Reporter1 will also not be allowed to access /trunk/Management\_Files because the Developer role, which is a "higher" role, has been explicitly denied access.

Whether a role is higher or not is determined by the position of the role in the **Roles and Permissions** page in the Administration page in Redmine.

# Installation #

## Prerequisites ##

First, make sure you can get Apache to authenticate using Redmine by following the instructions found [here](http://www.redmine.org/wiki/redmine/Repositories_access_control_with_apache_mod_dav_svn_and_mod_perl). If this doesn't work, then the plugin isn't going to work either.

All of the same prerequisites apply for the Redmine.pm module mentioned on Redmine's website.

## Installing the plugin ##

You can download the latest version of the Redmine Repository Control plugin using this command. Install the plugin using the directions found [here](http://www.redmine.org/wiki/redmine/Plugins).

```
svn checkout http://redminerepositorycontrol.googlecode.com/svn/trunk/ redmine_repository_control
```

## Configuring the plugin ##

We will be replacing Redmine.pm and changing the Apache configuration a bit. The rest of the configuration is done from within Redmine.

First, copy or link the RedmineRepoControl.pm file in the same way you did with Redmine.pm so that it is within perl's path. Next, modify your apache configuration to use our new module, use the following as an example. (notice the extra PerlAuthzHandler line, and the change from Redmine to RedmineRepoControl).

```
  PerlLoadModule Apache::RedmineRepoControl

   <Location /svn>
     DAV svn
     SVNParentPath "/var/svn" 

     AuthType Basic
     AuthName redmine
     Require valid-user

     PerlAccessHandler Apache::Authn::RedmineRepoControl::access_handler
     PerlAuthenHandler Apache::Authn::RedmineRepoControl::authen_handler
     PerlAuthzHandler Apache::Authn::RedmineRepoControl::authz_handler

     ## for mysql
     RedmineDSN "DBI:mysql:database=databasename;host=my.db.server" 
     ## for postgres
     # RedmineDSN "DBI:Pg:dbname=databasename;host=my.db.server" 

     RedmineDbUser "redmine" 
     RedmineDbPass "password" 
     #Cache the last 50 auth entries
     RedmineCacheCredsMax 50
  </Location>
```

Restart Apache, and make sure you don't have any errors. Now log into redmine and update your roles, making sure to add the **Manage Repository Controls** permission to the appropriate roles. Now add the **Repository Controls** module to a project, this should add a new **Repository Controls** tab to the project settings. At this point everything should be done and you can create controls.

# Using the Repository Controls plugin #

Once you add the **Repository Contrls** module to a project you will have a new tab in the project settings page. Selecting the **Repository Controls** tab will list all of the currently set controls, and allow you to remove and create roles.

When creating roles, you will be given a view of the repository with check boxes on the left side, and then below that you will see a selection for a Role and check boxes for access. You can select multiple entries in the repository, and it will create a new rule for each item you selected. Know that you do not need to select files deeper in the repository for it to be affected by the rule.