# Table of Contents
<details><summary>Click here to see the table of contents</summary><p>

- [Table of Contents](#table-of-contents)
- [Pre-Requisites](#pre-requisites)
- [Installing Gerrit in Ubuntu](#installing-gerrit-in-ubuntu)
    - [Java](#java)
        - [Installing Java](#installing-java)
    - [Create gerrit user in the system](#create-gerrit-user-in-the-system)
    - [Configuring gerrit](#configuring-gerrit)
        - [Creating a directory for store the files](#creating-a-directory-for-store-the-files)
        - [Getting gerrit war file](#getting-gerrit-war-file)
        - [Opening ports](#opening-ports)
        - [Initialize the site](#initialize-the-site)
        - [Creating SSH key for gerrit user](#creating-ssh-key-for-gerrit-user)
        - [Github plugin](#github-plugin)
            - [Build the github plugin](#build-the-github-plugin)
        - [Reconfiguration of gerrit](#reconfiguration-of-gerrit)
- [Replication plugin](#replication-plugin)
    - [Prerequisites](#prerequisites)
    - [from scratch](#from-scratch)
    - [Troubleshooting](#troubleshooting)
    - [From an existing repository from github.intel.com](#from-an-existing-repository-from-githubintelcom)
    - [Repository configuration](#repository-configuration)
- [Gerrit valuable advices](#gerrit-valuable-advices)
    - [Troubleshooting at start gerrit service](#troubleshooting-at-start-gerrit-service)
- [Setting project rules](#setting-project-rules)
    - [Submit Rule](#submit-rule)
        - [Setting project.config from All-Projects](#setting-projectconfig-from-all-projects)
    - [Remove the Verified category](#remove-the-verified-category)
        - [project.config](#projectconfig)
        - [rules.pl](#rulespl)
- [Setting up permissions for a repository](#setting-up-permissions-for-a-repository)
    - [Create a new group](#create-a-new-group)
    - [Project access](#project-access)
- [Plugins](#plugins)
    - [Importing a project through SSH connection](#importing-a-project-through-ssh-connection)
        - [Test SSH authentication](#test-ssh-authentication)
        - [Start importing the project](#start-importing-the-project)
- [Advices](#advices)
    - [Change the current Gerrit IP address](#change-the-current-gerrit-ip-address)
    - [Intel SSL Certificates](#intel-ssl-certificates)
        - [Getting a Intel Certificate](#getting-a-intel-certificate)
        - [Get CRT and KEY certificates](#get-crt-and-key-certificates)
    - [Configure Nginx as reverse proxy](#configure-nginx-as-reverse-proxy)
        - [Installing Nginx](#installing-nginx)
        - [Nginx Configuration File](#nginx-configuration-file)
        - [Gerrit Configuration File](#gerrit-configuration-file)
        - [Restarting services](#restarting-services)
</p></details>

  


# Pre-Requisites
<details><summary>Click here to see the pre-requisites</summary><p>

:notebook: This procedure was tested with the following configurations:

| Package/OS |      Version      |
| :--------: | :---------------: |
|   Ubuntu   |    16.04.4 LTS    |
|   gerrit   | gerrit-2.14.8.war |

The following packages are required by `Gerrit Code Review`:

```bash
# apt install gitweb
# apt  install git
```
</p></details>

# Installing Gerrit in Ubuntu

## Java

<details><summary>What will I get when I download Java software?</summary><p>

The Java Runtime Environment (JRE) is what you get when you download Java
software. The JRE consists of the Java Virtual Machine (JVM), Java platform
core classes, and supporting Java platform libraries. The JRE is the runtime
portion of Java software, which is all you need to run it in your Web browser
</p></details>

### Installing Java
If you do not have Java yet in your OS please install it with the following
command:

```bash
# apt update
# apt install default-jre
# apt install openjdk-8-jdk
```

<details><summary>How can I check the java version installed?</summary><p>

In order to check the java version installed in the OS please type the following
command

```bash
$ java -version
```
</p></details>

## Create gerrit user in the system

<details><summary>Click here to see the steps</summary><p>

You need to create a user for gerrit in the system with the following command:

```bash
# adduser gerrit

Adding user `gerrit' ...
Adding new group `gerrit' (1001) ...
Adding new user `gerrit' (1001) with group `gerrit' ...
Creating home directory `/home/gerrit' ...
Copying files from `/etc/skel' ...
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
Changing the user information for gerrit
Enter the new value, or press ENTER for the default
	Full Name []: Gerrit Code Review
	Room Number []:
	Work Phone []:
	Home Phone []:
	Other []:
Is the information correct? [Y/n] y
```
</p></details><br>

Change to the gerrit user previously created

```bash
$ sudo su gerrit
```


## Configuring gerrit

### Creating a directory for store the files
Create a directory in gerrit home directory to store all installation files:

```bash
$ mkdir -p /home/gerrit/packages
```

### Getting gerrit war file
Please download a gerrit.war from: 
[List of available gerrit releases][gerrit_releases]

Notice that this procedure was successful with **gerrit-2.14.8.war**

:warning: Please do not download any release that contains `rc` in their name,
since these release are **unstable**.

```bash
$ cd /home/gerrit/packages
$ wget --no-check-certificate https://gerrit-releases.storage.googleapis.com/gerrit-2.15.3.war/gerrit-2.14.8.war
```

### Opening ports

<details><summary>1. Enabling linux firewall</summary><p>

>  Ubuntu firewall is disabled by default

```bash
# ufw enable
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
```
</p></details>

<details><summary>2. Enabling SSH confections</summary><p>

The above command will disable the SSH connections due to the firewall, we
must proceed to enable it

- 2.1 List the applications configurations that `ufw` knows how to work with 

```bash
# ufw app list
Available applications:
  Apache
  Apache Full
  Apache Secure
  CUPS
  OpenSSH
```

- 2.2 Enable SSH connection by typing:

```
# ufw allow 'OpenSSH'
Rule added
Rule added (v6)
```
</p></details>

<details><summary>3. Opening the required ports</summary><p>

Make sure the linux `firewall` allows access to port 8081 and 29418 typing the
following commands in terminal:

```bash
# ufw allow 8081
Rule added
Rule added (v6)
# ufw allow 29418
Rule added
Rule added (v6)
# ufw allow 5900
Rule added
Rule added (v6)
# ufw allow 'Nginx Full'
# ufw status
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
8081                       ALLOW       Anywhere
29418                      ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
8081 (v6)                  ALLOW       Anywhere (v6)
29418 (v6)                 ALLOW       Anywhere (v6)
```

<details><summary>Click here to see port description</summary><p>

|    Port    | Description                             |
| :--------: | :-------------------------------------- |
|    8081    | Gerrit port                             |
|   29418    | Gerrit SSH port                         |
|    5900    | Port for VNC connections                |
| Nginx Full | nginx port for HTTP and HTTPS protocols |
</p></details>
</p></details><br>

📓 **If you are doing this behind a reverse proxy opening these ports is not
necessary, open only the ports the reverse proxy uses**

### Initialize the site

To configure gerrit in a interactive way, we have to type the following
command:

```bash
$ cd /home/gerrit/packages
$ java -jar <gerrit_file.war> init -d /home/gerrit/tree 
```
<details><summary>Click here to see the expected output</summary><p>

The above command will produce an output similar to this and it will enable to
use reverse-proxy for Gerrit, please take note of of the following values in
order to set it:

```text
Using secure store: com.google.gerrit.server.securestore.DefaultSecureStore
[2016-02-12 05:15:53,197] [main] INFO  com.google.gerrit.server.config.GerritServerConfigProvider : No /home/gerrit/gerrit/etc/gerrit.config; assuming defaults

*** Gerrit Code Review 2.14.8
*** 

Create '/home/gerrit/tree'   [Y/n]? Y

*** Git Repositories
*** 

Location of Git repositories   [git]: 

*** SQL Database
*** 

Database server type           [h2]: 

*** Index
*** 

Type                           [lucene/?]: 

*** User Authentication
*** 

Authentication method          [openid/?]: HTTP
Get username from custom HTTP header [y/N]? y
Username HTTP header           [SM_USER]: GITHUB_USER
SSO logout URL                 : 
Enable signed push support     [y/N]? 

*** Review Labels
*** 

Install Verified label         [y/N]? y

*** Email Delivery
*** 

SMTP server hostname           [localhost]: smtp.intel.com
SMTP server port               [(default)]: 
SMTP encryption                [none/?]: 
SMTP username                  [gerrit]: 
gerrit's password              : 
              confirm password : 

*** Container Process
*** 

Run as                         [gerrit]: 
Java runtime                   [/usr/lib/jvm/java-8-openjdk-amd64/jre]: 
Copy gerrit-2.14.8.war to /home/gerrit/gerrit/bin/gerrit.war [Y/n]? Y
Copying gerrit-2.14.8.war to /home/gerrit/gerrit/bin/gerrit.war

*** SSH Daemon
*** 

Listen on address              [*]: 
Listen on port                 [29418]: 
Generating SSH host key ... rsa... dsa... ed25519... ecdsa 256... ecdsa 384... ecdsa 521... done

*** HTTP Daemon
*** 

Behind reverse proxy           [y/N]? y
Use SSL (https://)             [y/N]? y
Subdirectory on proxy server   [/]:
Listen on address              [*]: 
Listen on port                 [8081]: 
Canonical URL                  [https://null]: https://<IP>/          

*** Cache
*** 

*** Plugins
*** 

Installing plugins.
Install plugin commit-message-length-validator version v2.14.8 [y/N]? y
Installed commit-message-length-validator v2.14.8
Install plugin download-commands version v2.14.8 [y/N]? y
Installed download-commands v2.14.8
Install plugin hooks version v2.14.8 [y/N]? y
Installed hooks v2.14.8
Install plugin replication version v2.14.8 [y/N]? y
Installed replication v2.14.8
Install plugin reviewnotes version v2.14.8 [y/N]? y
Installed reviewnotes v2.14.8
Install plugin singleusergroup version v2.14.8 [y/N]? y
Installed singleusergroup v2.14.8
Initializing plugins.

Initialized /home/gerrit/tree
Executing /home/gerrit/tree/bin/gerrit.sh start
Starting Gerrit Code Review: OK
Waiting for server on <IP>:<PORT> ... OK
Opening https://<IP>:<PORT>/#/admin/projects/ ...OK
```
</p></details><br>

:eyes: WARNING: `Canonical URL` should not have the Gerrit port since it will
be under a reverse-proxy (nginx), otherwise the github-plugin will fails when
users want to authenticate.

### Creating SSH key for gerrit user

We have to create a SSH key for gerrit user, please type the following command
in terminal:

```bash
$ ssh-keygen
```

<details><summary>Click here to see the output</summary><p>

```text
Generating public/private rsa key pair.
Enter file in which to save the key (/home/gerrit/.ssh/id_rsa):
Created directory '/home/gerrit/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/gerrit/.ssh/id_rsa.
Your public key has been saved in /home/gerrit/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:4vvf0sn+EgkbjYb6uxrpVBK8k1mHARjYahouz8zofYk gerrit@computing
The key's randomart image is:
+---[RSA 2048]----+
|   o.o...        |
|  . o.   o       |
|   .  o o..o     |
|. o    *..= .    |
|.+    B.S. + .   |
|o.   ..*  . o    |
|.*  . *.   o o   |
|. *E = o. ..=    |
|.. .. +o++.ooo.  |
+----[SHA256]-----+
```
</p></details><br>

:warning: Before to proceed please complete **CAREFULLY** the following steps:

1. [Getting a Intel Certificate](#getting-a-intel-certificate)
2. [Get CRT and KEY Certificates](#get-crt-and-key-certificates)
3. [Nginx Configuration File](#nginx-configuration-file)
4. [Gerrit Configuration File](#gerrit-configuration-file)



### Github plugin

The github plugin allows to connect gerrit with github for user authentication

In order to install it you must follow the following steps

1. Stop gerrit to install github plugin

```bash
$ /home/gerrit/tree/bin/gerrit.sh stop
```
<details><summary><b><i>Lazy installation</b></i></summary><p>

Gerrit has their own Jenkins build system for most of the plugins, please visit
the following link and search for the most recent version of `github-plugin`.

[Jenkins Gerrit](#jenkins_gerrit) 

:notebook: At this moment the version `2.14-SNAPSHOT` is the most recent.

<details><summary>Click here to see the steps</summary><p>

1. Installing `github-oauth.jar` file in gerrit tree

```bash
$ cd /home/gerrit/tree/lib
$ wget https://gerrit-ci.gerritforge.com/job/plugin-github-mvn-stable-2.14/lastSuccessfulBuild/artifact/github-oauth/target/github-oauth-2.14-SNAPSHOT.jar
```

2. Installing `github-plugin.jar` file in gerrit tree

```bash
$ cd /home/gerrit/tree/plugins
$ wget https://gerrit-ci.gerritforge.com/job/plugin-github-mvn-stable-2.14/lastSuccessfulBuild/artifact/github-plugin/target/github-plugin-2.14-SNAPSHOT.jar
```
</p></details>
</p></details>

<details><summary><b><i>Build from scratch</b></i></summary><p>

1. Clone the github plugin repository in the gerrit_source directory. Make sure
    to clone a stable branch:

```bash
$ cd /home/gerrit/packages
$ git clone -b stable-2.14 https://gerrit.googlesource.com/plugins/github
```

:notebook: For the following step you need to have maven installed.

<details><summary>Maven installation</summary><p>

To build the github plugin we need maven package installed in the system.
Maven is a command-line tool for building Java applications.

1. Installing maven package

:link: [maven environment setup][maven_env]

2. Configure proxies for maven

:link: [How do I use maven through a proxy][maven_proxy]

<details><summary>Click here to see the example</summary><p>

settings file is under: `/usr/local/apache-maven-3.5.3/conf/settings.xml`

```html
  <proxies>
    <!-- proxy
     | Specification for one proxy, to be used in connecting to the network.
     |-->
    <proxy>
      <id>optional</id>
      <active>true</active>
      <protocol>http</protocol>
      <host>proxy-fm.intel.com</host>
      <port>911</port>
      <nonProxyHosts>gerritcodereview.intel.com|www.google.com</nonProxyHosts>
    </proxy>
  </proxies>
```
</p></details>
</p></details>

##### Build the github plugin
In order to use maven to build the github plugin please type the following
commands: 

```bash
$ cd /home/gerrit/packages/github
$ mvn install
```

Copy the oauth and plugin jars generated by the maven build into the appropiate
directories in the gerrit installation path:

```bash
$ cp github-oauth/target/github-oauth-2.14-SNAPSHOT.jar /home/gerrit/tree/lib/
$ cp github-plugin/target/github-plugin-2.14-SNAPSHOT.jar /home/gerrit/tree/plugins/
```
</p></details>

#### Reconfiguration of gerrit 
To install the github plugin we have to re-run the gerrit initialization to install the plugins and configure them:

```bash
$ cd /home/gerrit/packages
$ java -jar <gerrit_file.war> init -d /home/gerrit/tree
```
<details><summary>Please type Enter until you are in this part:</summary><p>

```text
*** GitHub Integration***
 
GitHub URL                     [https://github.com]: https://github.intel.com
GitHub API URL                 [https://api.github.com]: https://github.intel.com/api/v3
 
NOTE: You might need to configure a proxy using http.proxy if you run Gerrit behind a firewall.
 
*** GitHub OAuth registration and credentials***
 
Register Gerrit as GitHub application on:
https://github.intel.com/settings/applications/new
 
Settings (assumed Gerrit URL: http://<IP>/)
* Application name: Gerrit Code Review
* Homepage URL: http://<IP>/
* Authorization callback URL: http://<IP>/oauth
 
After registration is complete, enter the generated OAuth credentials:
GitHub Client ID               : 0d7bd7f89c3bf8e9102c
GitHub Client Secret           :
              confirm password :
Gerrit OAuth implementation    [HTTP/?]:
HTTP Authentication Header     [GITHUB_USER]:
 
Initialized /home/gerrit/tree

```
</p></details><br>

:eyes: **WARNING**: the `Homepage URL` and the `Authorization callback URL` in [https://github.intel.com/settings/applications](https://github.intel.com/settings/applications) they should not have the Gerrit port since they will be behind a reverse-proxy (nginx), otherwise github-plugin will fails when users want to authenticate in Gerrit site.

Before starting gerrit again open `/home/gerrit/tree/etc/gerrit.config` and make sure the config looks similar to this, make the appropriate changes if necessary.

<details><summary>Click here to see gerrit.config file</summary><p>

```text
[gerrit]
        basePath = git
        serverId = 0d40f2c8-16b2-4870-852f-442dab914008
        canonicalWebUrl = https://<IP>/
[database]
       type = h2
        database = /home/gerrit/gerrit/db/ReviewDB
[index]
        type = LUCENE
[auth]
        type = HTTP
        httpHeader = GITHUB_USER
        httpExternalIdHeader = GITHUB_OAUTH_TOKEN
        loginUrl = /gerrit/login
        loginText = Sign-in with GitHub
        registerPageUrl = "/gerrit/#/register"
[receive]
        enableSignedPush = false
[sendemail]
        smtpServer = smtp.intel.com
[container]
        user = gerrit
        javaHome = /usr/lib/jvm/java-8-openjdk-amd64/jre
[sshd]
        listenAddress = *:29418
[httpd]
        listenUrl = proxy-https://127.0.0.1:8081/
        filterClass = com.googlesource.gerrit.plugins.github.oauth.OAuthFilter
[cache]
        directory = cache
[github]
        url = https://github.intel.com
        apiUrl = https://github.intel.com/api/v3
        clientId = 0d7bd7f89c3bf8e9102c
```
</p></details><br>

:eyes: `Add the SSH key from the gerrit user to the authorized SSH keys of one of the members of the github organization that will host the repositories.`
Make sure that the user has write permissions to the repositories.<br>

Add `github.intel.com` to the known_hosts of the gerrit user

```bash
$ ssh git@github.intel.com
The authenticity of host 'github.intel.com (GITHUB_IP)' can't be established.
ECDSA key fingerprint is SHA256:+.....
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'github.intel.com,GITHUB_IP' (ECDSA) to the list of known hosts.
PTY allocation request failed on channel 0
Hi GITHUB_USER! You've successfully authenticated, but GitHub does not provide shell access.
Connection to github.intel.com closed
```

Add a section for `github.intel.com` to `~/.ssh/config` file specifying the key
to use for the connection: id_rsa.<br>
Add the following to the file

```text
$ touch /home/gerrit/.ssh/config
$ chmod 600 /home/gerrit/.ssh/config
$ vim /home/gerrit/.ssh/config
Host github.intel.com
    IdentityFile /home/gerrit/.ssh/id_rsa
    PreferredAuthentications publickey
    StrictHostKeyChecking No
```

## Replication plugin
This plugin can automatically push any changes Gerrit Code Review makes to its
managed Git repositories to another system

### Prerequisites

1. Replication configuration file

Create the configuration file for the Github replication plugin

```bash
$ vim /home/gerrit/tree/etc/replication.config

[remote "github"]  
    url = git@github.intel.com:<my_organization>/${name}.git
```

This configuration enables all of the repositories in Gerrit to be replicated
to GitHub under the my organization GitHub Team account.


2. Start the Gerrit server

```bash
$ /home/gerrit/tree/bin/gerrit.sh start
```

:notebook: Check the logs into `/home/gerrit/tree/logs` for errors, there
should be no errors at this point


Login to Gerrit home page using the canonical URL and register your user
`The first user to login to Gerrit becomes the admin user`

### from scratch
This section will covers the configuration from scratch. This mean that the user
will create a new repository into (github.intel.com)[github.intel.com] 

1. Create a new repository into [github.intel.com](https://github.intel.com)

<details><summary>Click here to see the image</summary><p>

![alt text][create_repo_github]
</p></details><br>

:warning: **Never initialize the repository from GitHub with an empty commit or
readme file; otherwise the first replication attempt from Gerrit will result
in a conflict and will then fail**.

2. Clone the repository previously created into some workspace, e.g: `/tmp`

```bash
$ git -C /tmp clone https://github.intel.com/<my_organization>/my_repository.git
```

3. Create a repository in gerrit
This repository will be the master repository hosted in gerrit that will be the
master repository that will push the changes to `github.intel.com`<br>

<details><summary>Please use the following image as an example</summary><p>

![alt text][create_repo_gerrit]
</p></details><br>

:notebook: In order to avoid issues with `git` please select click on the option
`Create initial empty commit`

As soon as it is created the Gerrit's repository a commit will be available into
`github.intel.com/<my_organization>/my_repository`, please go and check it.

### Troubleshooting
At this point replication log located into `/home/gerrit/gerrit/logs/replication_log`
will have some information available to show.<br>
If any error occurs please check the logs in the folder.

### From an existing repository from github.intel.com

1. Clone the existing repository from `github` in some workspace, eg:

```bash
$ git -C /tmp clone --bare <my_existing_repository.git>
```

2. Create a repository in Gerrit

You have to create a new repository in Gerrit with the same name as appears in
`github.intel.com`

<details><summary>Click here to see the image</summary><p>

![all_text][create_repo_gerrit_b]
</p></details><br>

:notebook: Please **DO NOT CLICK** in the options `Create initial empty commit`
and/or `Only server as parent for others projects` since this will be a an
existing repository

3. Synchronize Gerrit and Github repository 

```bash
$ git -C /tmp push ssh://<GITHUB_USER>@<GERRIT_HOST>:29418/<EXISTING_REPO_IN_GERRIT>.git *:*
```

### Repository configuration

1. Create `.gitreview` file

`gitreview` file is needed in the repositories in order to configure the commits
towards Gerrit

Please clone the repository from Github/Gerrit in some workspace and create the
file with the following information and change the appropriate variables

:warning: Do not clone it with `--bare` option

```bash
$ vim .gitreview
[gerrit]
host=<GERRIT_HOST>
port=29418
project=<REPOSITORY_NAME>
defaultbranch=master 
```

2. Configuring Gerrit in the repository

`git-review` package is needed in order to configure `.gerritreview` file in the
repository, please follow the next steps:

```bash
$ sudo apt-get install git-review
$ git review -s
Creating a git remote called "gerrit" that maps to:
	ssh://hiperezr@10.219.128.134:29418/my_project
```

To corroborate that gerrit is well configured the repository must have a remote
repository called `gerrit`, to check this please type in terminal:

```bash
$ git remote -v
gerrit	ssh://hiperezr@10.219.128.134:29418/my_project (fetch)
gerrit	ssh://hiperezr@10.219.128.134:29418/my_project (push)
origin	https://github.intel.com/androidgraphics/my_project.git (fetch)
origin	https://github.intel.com/androidgraphics/my_project.git (push)
```

:notebook: The `origin` repository is the default provided by `git`

3. Uploading gitreview file

The following step will skips the Gerrit's normal work flow and this step only
can be performed by an Gerrit' admin and with the only purpose of making the
initial configuration

```bash
$ git add .gitreview
$ git commit -m "some commit information"
$ git push gerrit
```

:notebook: You will notice in a few seconds that this change will be reflected
in the following two spaces:

- github.intel.com
- gerrit internal repository `/home/gerrit/gerrit/git/my_repository.git`


# Gerrit valuable advices

<details><summary>How to restart Gerrit</summary><p>

On some occasions we will see the need to restart the gerrit service, e.g:
install a new gerrit plugin, apply some specific configuration, etc.
Please see the following commands:

1. stop gerrit service

```bash
$ /home/gerrit/gerrit/bin/gerrit.sh stop 
```

2. start gerrit service

```bash
$ /home/gerrit/gerrit/bin/gerrit.sh start 
```

3. restart gerrit service

```bash
$ /home/gerrit/gerrit/bin/gerrit.sh restart 
```
</p></details>

### Troubleshooting at start gerrit service
TBD


# Setting project rules


## Submit Rule
A Submit Rule in Gerrit is logic that defines when a change is submittable.
By default, a change is submittable when it gets at least one highest vote in
each voting category and has no lowest vote (aka veto vote) in any category.
Typically, this means that a change needs Code-Review+2, Verified+1 and has
neither Code-Review-2 nor Verified-1 to become submittable.


> More examples in [Prolog Cookbook](
https://gerrit-review.googlesource.com/Documentation/prolog-cookbook.html)

<details><summary>Example 1</summary><p>

```text
% NOTE :  the first rule does not works with the second one

% rule:     : No Self Approvals (Code-Review +2)
% rationale : deny any changes submittable from the author himself

%submit_rule(S) :-                                                       
%    gerrit:default_submit(X),                                           
%    X =.. [submit | Ls],                                                
%    add_non_author_approval(Ls, R),                                     
%    S =.. [submit | R].

add_non_author_approval(S1, S2) :-
    gerrit:commit_author(A),
    gerrit:commit_label(label('Code-Review', 2), R),
    gerrit:commit_label(label('Code-Review', 1), R),
    R \= A, !,
    S2 = [label('Non-Author-Code-Review', ok(R)) | S1].

add_non_author_approval(S1, [label('Non-Author-Code-Review', need(_)) | S1]).

% rule      : 1+1+1+1=4 | 2+2=4 | 2+1+1=4 (Code-Review)
% rationale : introduce accumulative voting to determine if a change
%             is submittable or not and make the change submittable
%             if the total score is 4 or higher.

sum_list([], 0).
sum_list([H | Rest], Sum) :- sum_list(Rest,Tmp), Sum is H + Tmp.

add_category_min_score(In, Category, Min,  P) :-
    findall(X, gerrit:commit_label(label(Category,X),R),Z),
    sum_list(Z, Sum),
    Sum >= Min, !,
    gerrit:commit_label(label(Category, V), U),
    V >= 1,
    !,
    P = [label(Category,ok(U)) | In].

add_category_min_score(In, Category,Min,P) :-
    P = [label(Category,need(Min)) | In].

submit_rule(S) :-
    gerrit:default_submit(X),
    X =.. [submit | Ls],
    gerrit:remove_label(Ls,label('Code-Review',_),NoCR),
    add_category_min_score(NoCR,'Code-Review', 4, Labels),
    S =.. [submit | Labels].
```
</p></details>

<details><summary>Example 2</summary><p>

```text
% ====================================================================
% rule:        : No Self Approvals (Code-Review +1 +2)
% rationale    : deny any changes submittable from the author himself
% how to works : if the author grants himself any qualification either 
%                +1 or +2 the patch automatically will be blocked until
%                the author remove their qualification
% ====================================================================

% make the qualification of +2 invalid
submit_rule(submit(CR, V, N)) :-
  gerrit:commit_author(A),
  gerrit:max_with_block(-2, 2, 'Code-Review', label(_, ok(A))),
  N = label('Non-Author Code-Review', need(_)),
  base(CR, V),
  !.
submit_rule(submit(CR, V)) :-
  base(CR, V).
base(CR, V) :-
  gerrit:max_with_block(-1, 1, 'Verified', V),
  gerrit:max_with_block(-2, 2, 'Code-Review', CR).


% make the qualification of +1 invalid
submit_rule(submit(CR, V, N)) :-
  gerrit:commit_author(A),
  gerrit:max_with_block(-1, 1, 'Code-Review', label(_, ok(A))),
  N = label('Non-Author Code-Review', need(_)),
  base(CR, V),
  !.
submit_rule(submit(CR, V)) :-
  base(CR, V).
base(CR, V) :-
  gerrit:max_with_block(-1, 1, 'Verified', V),
  gerrit:max_with_block(-1, 1, 'Code-Review', CR).



% ================================================================
% rule      : label verified as optional
% rationale : label verified is used for an automated system such
%             as Jenkis in order to grant -1, +1 qualification for
%             a job performed in the patch like code analysis.
% ================================================================

submit_rule(submit(CR)) :-
  gerrit:commit_message_matches('^Verified-by: '),
  !,
  gerrit:max_with_block(-2, 2, 'Code-Review', CR).
submit_rule(submit(CR, V)) :-
  gerrit:max_with_block(-1, 1, 'Verified', V),
  gerrit:max_with_block(-2, 2, 'Code-Review', CR).



% ================================================================
% rule         : sum of points, 1+1+1=3 | 2+1=3 (Code-Review)
% how to works : Gerrit needs to have 3 points at least in total in
%                order to be submittable.
% ================================================================

sum_list([], 0).
sum_list([H | Rest], Sum) :- sum_list(Rest,Tmp), Sum is H + Tmp.

add_category_min_score(In, Category, Min,  P) :-
    findall(X, gerrit:commit_label(label(Category,X),R),Z),
    sum_list(Z, Sum),
    Sum >= Min, !,
    gerrit:commit_label(label(Category, V), U),
    V >= 1,
    !,
    P = [label(Category,ok(U)) | In].

add_category_min_score(In, Category,Min,P) :-
    P = [label(Category,need(Min)) | In].

submit_rule(S) :-
    gerrit:default_submit(X),
    X =.. [submit | Ls],
    gerrit:remove_label(Ls,label('Code-Review',_),NoCR),
    add_category_min_score(NoCR,'Code-Review', 3, Labels),
    S =.. [submit | Labels].
```
</p></details>

<details><summary>Example 3</summary><p>

This prolog rule works with verified vote from Jenkins

```text
sum_list([], 0).
sum_list([H | Rest], Sum) :- sum_list(Rest,Tmp), Sum is H + Tmp.

add_category_min_score(In, Category, Min,  P) :-
    findall(X, gerrit:commit_label(label(Category,X),R),Z),
    sum_list(Z, Sum),
    Sum >= Min, !,
    gerrit:commit_label(label(Category, V), U),
    V >= 1,
    !,
    P = [label(Category,ok(U)) | In].

add_category_min_score(In, Category,Min,P) :-
    P = [label(Category,need(Min)) | In].

submit_rule(S) :-
    gerrit:default_submit(X),
    X =.. [submit | Ls],
    gerrit:remove_label(Ls,label('Code-Review',_),NoCR),
    add_category_min_score(NoCR,'Code-Review', 2, Labels),
    S =.. [submit | Labels].
```


</p></details>

### Setting project.config from All-Projects

To download the `project.config`|`groups` from `All-Projects` do the following:

```bash
$ git clone ssh://GERRIT_SERVER:29418/All-Projects
# EDIT project.config
$ git add YOUR_FILES
$ git commit
$ git push origin HEAD:refs/meta/config
```

<details><summary><b><i>groups</b></i> file</summary><p>

```text
% cat groups
# UUID                                  	Group Name
#
1ca6bf8472c1de9e8758353597bde580c8bf5a78	STX-QE-Submitters
2070f76ae64905a0d277a255a569f248de46b0a6	stx-test-suite-branch-stage-1-core
48a349a926f3d2912ec6876e9d471a3bd27d2ff9	Non-Interactive Users
6f5e7bbc51a19b75a831b205d549024ac7f5be78	stx-test-suite-branch-stage-1
77dfd0e12754c3dd19b7013d73142a59ae83f979	Administrators
95579f78517c465202249af7bf7269e7d5cb918f	STX-QE-Contributors
9a9253ef44f445df0bc5250d3c3ee0b75042c513	stx-test-suite-branch-master-next
c69ac882294024193411ce3c8c7f08905fe0d0bc	stx-test-suite-branch-master-next-core
baf902a86f9c7f8a1eacd9dab797b611c0c77618	STX-QE-Core
bd040f02ca5ddb88087d8c0c4a6d6012090ba293	stx-test-suite-branch-configuration-support
d58839a0dc2666f7462ca0d1fa090185517995bd	stx-test-suite-branch-configuration-support-core
cb37776aca474e9f0b0445f404b6347e3a3ad459	stx-test-suite-branch-master
4f15e2007647cbacd2fd4debb102c2258d4525f6	stx-test-suite-branch-master-core
cc7cd6ff01f339a231459f4b649c14c38c5b87aa	stx-test-suite-branch-baremetal
c7331a62e7aefe4a2086f84efc03be3dd4b9c1b8	stx-test-suite-branch-baremetal-core
global:Change-Owner                     	Change Owner
global:Project-Owners                   	Project Owners
global:Registered-Users                 	Registered Users
```
> `groups` file must have a tab as separator
</p></details>

<details><summary><b><i>project.config</b></i> file</summary><p>

The following configuration enables ...

- Custom permissions of `label-Code-Review` on different branches
- Block -2,-1,+1,+2 for the owner in the label `label-Code-Review` in the
section `[access "refs/*"]`
- restricted access to whom can be cloned a specific branches in the project
`stx-test-suite`

> Warning 1: this only works if the changes will be pushed from SSH, not in the
Gerrit UI<br>
Warning 2: this changes affects globally to `All-Projects` this mean that could
not be two projects with the same branch name, in this case this kind of
settings needs to be in each project into `project.config` > 
`[access "refs/heads/configuration_support"]`, but this is pure theory I have
not tried the changes. 

```text
[project]
	description = Access inherited by all other projects.
[receive]
	requireContributorAgreement = false
	requireSignedOffBy = false
	requireChangeId = true
	enableSignedPush = false
[submit]
	mergeContent = true
[capability]
	administrateServer = group Administrators
	priority = batch group Non-Interactive Users
	streamEvents = group Non-Interactive Users
	streamEvents = group Event Streaming Users
	createAccount = group Administrators
	modifyAccount = group Administrators
[access "refs/*"]
	read = group Administrators
	read = group Non-Interactive Users
	read = group Registered Users
    label-Code-Review = block -2..+2 group Change Owner
    label-Code-Review = block -1..+1 group Change Owner
    create = group Administrators
[access "refs/heads/*"]
	create = group Administrators
	create = group Project Owners
	create = group STX-QE-Core
	forgeAuthor = group Administrators
	forgeCommitter = group Administrators
	forgeCommitter = group Project Owners
	label-Code-Review = -2..+2 group Project Owners
	label-Code-Review = -1..+1 group Non-Interactive Users
	editTopicName = +force group Administrators
	editTopicName = +force group Project Owners
	editTopicName = group STX-QE-Core
	abandon = group Registered Users
	label-Verified = -1..+1 group Non-Interactive Users
	viewDrafts = group Administrators
	editAssignee = group STX-QE-Core
	deleteDrafts = group Administrators
	removeReviewer = group STX-QE-Core
	submit = group Administrators
	submit = group Project Owners
	submit = group STX-QE-Submitters
	submitAs = group STX-QE-Core
	publishDrafts = group STX-QE-Contributors
	publishDrafts = group STX-QE-Core
	push = group Administrators
    pushMerge = group Administrators
[access "refs/meta/config"]
	exclusiveGroupPermissions = read
	read = group Administrators
	read = group Project Owners
	create = group Administrators
	create = group Project Owners
	push = group Administrators
	push = group Project Owners
	label-Code-Review = -2..+2 group Administrators
	label-Code-Review = -2..+2 group Project Owners
	label-Code-Review = -2..+2 group STX-QE-Core
	label-Code-Review = -1..+1 group STX-QE-Contributors
	submit = group Administrators
	submit = group Project Owners
	submit = group STX-QE-Core
[access "refs/tags/*"]
	create = group Administrators
	create = group Project Owners
	createTag = group Administrators
	createTag = group Project Owners
	createSignedTag = group Administrators
	createSignedTag = group Project Owners
[label "Code-Review"]
	function = MaxWithBlock
	defaultValue = 0
	copyMinScore = true
	copyAllScoresOnTrivialRebase = true
    copyAllScoresIfNoCodeChange = true
    copyAllScoresIfNoChange = true
	value = -2 Ohh, hell no!
	value = -1 Hmm, I'm not a fan
	value =  0 No score, only a simple comment
	value = +1 I like, but need another to like it as well
	value = +2 Ohh, hell yes!
[label "Verified"]
	function = MaxWithBlock
	value = -1 Fails
	value =  0 No score
	value = +1 Verified
	copyAllScoresIfNoCodeChange = true
	defaultValue = 0
[access "refs/for/*"]
	addPatchSet = group Registered Users
	create = group Administrators
[access "refs/for/refs/heads/baremetal"]
    addPatchSet = group stx-test-suite-branch-baremetal-core
	addPatchSet = group stx-test-suite-branch-baremetal
	push = group stx-test-suite-branch-baremetal-core
    push = group stx-test-suite-branch-baremetal
    pushMerge = group stx-test-suite-branch-baremetal-core
	pushMerge = group stx-test-suite-branch-baremetal
	create = group Administrators
[access "refs/for/refs/heads/master"]
    addPatchSet = group stx-test-suite-branch-master-core
	addPatchSet = group stx-test-suite-branch-master
    push = group stx-test-suite-branch-master-core
	push = group stx-test-suite-branch-master
    pushMerge = group stx-test-suite-branch-master-core
	pushMerge = group stx-test-suite-branch-master
	create = group Administrators
[access "refs/for/refs/heads/master_next"]
    addPatchSet = group stx-test-suite-branch-master-next-core
	addPatchSet = group stx-test-suite-branch-master-next
    push = group stx-test-suite-branch-master-next-core
	push = group stx-test-suite-branch-master-next
    pushMerge = group stx-test-suite-branch-master-next-core
	pushMerge = group stx-test-suite-branch-master-next
	create = group Administrators
[access "refs/for/refs/heads/configuration_support"]
    addPatchSet = group stx-test-suite-branch-configuration-support-core
	addPatchSet = group stx-test-suite-branch-configuration-support
    push = group stx-test-suite-branch-configuration-support-core
	push = group stx-test-suite-branch-configuration-support
    pushMerge = group stx-test-suite-branch-configuration-support-core
	pushMerge = group stx-test-suite-branch-configuration-support
	create = group Administrators
[access "refs/for/refs/heads/stage_1"]
    addPatchSet = group stx-test-suite-branch-stage-1-core
	addPatchSet = group stx-test-suite-branch-stage-1
    push = group stx-test-suite-branch-stage-1-core
	push = group stx-test-suite-branch-stage-1
    pushMerge = group stx-test-suite-branch-stage-1-core
	pushMerge = group stx-test-suite-branch-stage-1
	create = group Administrators
[access "refs/heads/master"]
	label-Code-Review = -2..+2 group stx-test-suite-branch-master-core
	label-Code-Review = -1..+1 group stx-test-suite-branch-master
	create = group Administrators
[access "refs/heads/master_next"]
	label-Code-Review = -2..+2 group stx-test-suite-branch-master-core
	label-Code-Review = -1..+1 group stx-test-suite-branch-master
	create = group Administrators
[access "refs/heads/baremetal"]
	label-Code-Review = -2..+2 group stx-test-suite-branch-baremetal-core
    label-Code-Review = -1..+1 group stx-test-suite-branch-baremetal
    create = group Administrators
[access "refs/heads/configuration_support"]
	label-Code-Review = -2..+2 group stx-test-suite-branch-configuration-support-core
	label-Code-Review = -2..+2 group Non-Interactive Users
    label-Code-Review = -1..+1 group stx-test-suite-branch-configuration-support
    create = group Administrators
[access "refs/heads/stage_1"]
	label-Code-Review = -2..+2 group stx-test-suite-branch-stage-1-core
	label-Code-Review = -1..+1 group stx-test-suite-branch-stage-1
	create = group Administrators
```
</p></details>

## Remove the Verified category
A project has no build and test. It consists of only text files and needs only
code review. We want to remove the Verified category from this project so that
Code-Review+2 is the only criteria for a change to become submittable.
We also want the UI to not show the Verified category in the table with votes
and on the voting screen.

1. Clone the repository 

```bash
$ git clone <GITHUB_PROJECT>
```

2. Add a [gitreview file](#repository-configuration)

3. Setup `gerrit` as remote

```bash
$ git review -s
```

4. Checkout the config branch in the `github` repository

```bash
$ git fetch gerrit refs/meta/config:config
$ git checkout config
# ... edit or create the rules.pl file
# ... edit or create the project.config file
$ git add rules.pl
$ git commit -m "My submit rules"
$ git push gerrit HEAD:refs/meta/config
```

### project.config

```bash
$ vim project.config
[access]
	inheritFrom = All-Projects
[label "Verified"]
	defaultValue = 0
	value =  0 No score
	function = NoBlock
```

### rules.pl

```bash
$ vim rules.pl
submit_rule(submit(CR)) :-
    gerrit:max_with_block(-2, 2, 'Code-Review', CR).
```

# Setting up permissions for a repository

## Create a new group
Go to People > Create New Group

<details><summary>Click here to see the image</summary><p>

![all_text][new_group]
</p></details>

## Project access
Go to Projects > access as the following image

<details><summary>Click here to see the image</summary><p>

![all_text][project_access]
</p></details>

In the above example we have created the group called `STX-QE-integratos`, please
set up the permission for that group as appears in the following images:

<details><summary>Click here to see the image</summary><p>

![all_text][project_permissions_a]
![all_text][project_permissions_b]
![all_text][project_permissions_c]
![all_text][project_permissions_d]
</p></details>

# Plugins

The Gerrit server functionality can be extended by installing plugins.

<details><summary>Importer plugin</summary><p>

The importer plugin allows to import projects from one Gerrit server into another Gerrit server

Projects can be imported while both source and target Gerrit server are online. There is no downtime required.

The git repository and all changes of the project, including approvals and review comments, are imported. Historic timestamps are preserved.

Project imports can be resumed. This means a project team can continue to work in the source system while the import to the target system is done. By resuming the import the project in the target system can be updated with the missing delta.

The importer plugin can also be used to copy a project within one Gerrit server, and in combination with the [delete-project](https://review.openstack.org/Documentation/config-plugins.html#delete-project) plugin it can be used to rename a project.

[Project](https://gerrit-review.googlesource.com/#/admin/projects/plugins/importer) | [Documentation](https://gerrit.googlesource.com/plugins/importer/+doc/master/src/main/resources/Documentation/about.md)



### Importing a project through SSH connection

#### Test SSH authentication

Before to proceed importing project, we need to ensure we are able to comunicate with Gerrit "source/destionation" typing the follwoing command:

```bash
$ ssh -p 29418 <gerrit_user>@<gerrit_ip>

  ****    Welcome to Gerrit Code Review    ****

  Hi Gerrit User, you have successfully connected over SSH.

  Unfortunately, interactive shells are disabled.
  To clone a hosted Git repository, use:

  git clone ssh://<gerrit_user>@<gerrit_ip>:29418/REPOSITORY_NAME.git

Connection to <gerrit_ip> closed.
```

If you are not able to perform the above step please consult the following link:

[Troubleshooting](https://gerrit-review.googlesource.com/Documentation/error-permission-denied.html)

#### Start importing the project

In the following example we must suppose the following:

1. You are in the gerrit destination host

```bash
$ ssh -p 29418 <gerrit_user>@<gerrit_ip> importer project --from http://<gerrit_external_ip>:8080 --pass <http_password_from_external_gerrit> --user <gerrit_external_user> <gerrit_external_project_name>
```

<details><summary>Click here to see the table</summary><p>


| Variable                            | Explanation                                                  |
| ----------------------------------- | ------------------------------------------------------------ |
| gerrit_user                         | The current gerrit user for the destination host             |
| gerrit_ip                           | The current gerrit ip for the destination host               |
| gerrit_external_ip                  | The gerrit ip from where the project(s) will be imported     |
| http_password_from_external_gerrrit | This password will generate the source gerrit under: User > Settings > HTTP Password > Generate Password |
| gerrit_external_user                | The gerrrit user from the source where the project will be imported |
| gerrit_external_project_name        | The gerrit project form the source that will be imported     |
</p></details>
</p></details>


# Advices

This section contains useful advices for some common question related Gerrit configuration

## Change the current Gerrit IP address

If for some reason we have to change the current Gerrit IP address please follow the next steps

1. Stop Gerrit server  `/home/gerrit/tree/bin/gerrit.sh stop`
2. goto `/home/gerrit/tree/etc/gerrit.config` and change the **canonicalWebUrl** for the desire IP address

If you have Github plugin enable the following step is required

1. Goto [https://github.intel.com/settings/developers](https://github.intel.com/settings/developers) and select your Gerrit application in order to change **Homepage URL** and **Authorization callback URL** value for the desire IP address

2. Rebuild gerrit.war in order to take the values from gerrit.config

   ```bash
   $ cd /home/gerrit/packages
   $ java -jar <gerrit_file.war> init -d /home/gerrit/tree 
   ```

Start Gerrit server again `/home/gerrit/tree/bin/gerrit.sh start`



## Intel SSL Certificates

### Getting a Intel Certificate

An organization needs to install the SSL Certificate onto its web server to initiate a secure session with browsers. Once a secure connection is established, all web traffic between the web server and the web browser will be secure.

When a certificate is successfully installed on your server, the application protocol (also known as HTTP) will change to HTTPs, where the **S** stands for **secure**



1. Goto [https://certificates.intel.com/PkiWeb](https://certificates.intel.com/PkiWeb)
2. Select `Request a certificate` > `Intel SSL SHA2`



The step 2 will redirect to Intel process for requesting a SSL Certificate.

:warning: Please notice that the certificate name should be match with the Gerrit IP or a CNAME, e.g `gerritcodereview.intel.com`



> Usually the certificate getting from Intel is a `pfx` file



### Get CRT and KEY certificates

The certificate from Intel coming in a `pfx` format file **"PKCS#12"** that contains the `crt` and `key` certificates, ngnix needs to have them in single files.



1. Getting `key` certificate

2. ```bash
   $ openssl pkcs12 -in [certificate.pfx] -nocerts -out certificate_with_password.key
   Enter Import Password:
   MAC verified OK
   Enter PEM pass phrase:
   Verifying - Enter PEM pass phrase:
   ```

 - `Enter Import Password` = The password you put when created the certificated

 - `Enter PEM pass phrase` = The phrase to unlock the certificate

   

2. Getting `crt` certificate

1. ```bash
   $ openssl pkcs12 -in [certificate.pfx] -clcerts -nokeys -out certificate.crt
   Enter Import Password:
   MAC verified OK
   ```

- `Enter Import Password` = The password you put when created the certificated



3. Generate a certificate.key without password

We have to generate a new certificate.key without password in order to use with
nginx

```bash
$ openssl rsa -in certificate_with_password.key] -out certificate.key
```

4. Copy both certificates to `/etc/nginx/conf.d` in Gerrit host

## Configure Nginx as reverse proxy

**NGINX** is open source software for web serving, reverse proxying, caching,
load balancing, media streaming, and more. It started out as a web server
designed for maximum performance and stability. In addition to its HTTP server
capabilities, NGINX can also function as a proxy server for email (IMAP, POP3,
and SMTP) and a reverse proxy and load balancer for HTTP, TCP, and UDP servers.

<details><summary>Why do i need a reverse proxy?</summary><p>

This question is simply to answer when you are care about the security.
**Nginx** will offers the following functionalities:

1.  Offer a secure HTTPS connection

2. Hide Gerrit port and connect directly with the IP/Hostname
</p></details>
   

### Installing Nginx

```bash
# apt-get install nginx
```



### Nginx Configuration File

<details><summary>Click here to see the configuration</summary><p>

**Please change the required fields** with `<>`

```bash
# vim /etc/nginx/sites-available/gerritcodereview

server {
        listen 80;
        server_name <IP or CNAME>;
        return 301 https://$host$request_uri;
}
server {
        listen 443;
        server_name <IP or CNAME>;

		ssl on;
		ssl_certificate conf.d/certificate.crt;
		ssl_certificate_key conf.d/certificate.key;

        location ^~ / {
                proxy_pass        http://127.0.0.1:8081;
                proxy_set_header  X-Forwarded-For $remote_addr;
                proxy_set_header  Host $host;
        }
}
```

> The above nginx configuration for Gerrit site will redirect all the connections `HTTP` to `HTTPS` to force to use the certificates.
</p></details><br>

Enable the ngnix configuration file

```bash
# ln -s /etc/nginx/sites-available/gerritcodereview /etc/nginx/sites-enabled/
```

Restart nginx

```bash
# service nginx restart
```

### Gerrit Configuration File

Gerrit also needs of a configuration to run with the reverse proxy, only the
following part concerns to this configuration

```bash
[gerrit]
	canonicalWebUrl = http://<gerrit_ip_address>:8081/
[httpd]
	listenUrl = proxy-https://127.0.0.1:8081/
```

> `gerrit_ip_address` also can be a CNAME, e.g : `gerritcodereview.intel.com`

### Restarting services

```bash
# service nginx restart
```

```bash
$ /home/gerrit/tree/bin/gerrit.sh restart
Stopping Gerrit Code Review: OK
Starting Gerrit Code Review: OK
```



Now, you can go to the following url to check this: `https://<gerrit_ip>`

<details><summary>Click here to see the references</summary><p>

:link: [How to install java on ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-install-java-with-apt-get-on-ubuntu-16-04)<br>
:link: [Using Gerrit-Github](https://hub.packtpub.com/using-gerrit-github/)<br>
:link: [Ubuntu 16.04 Gerrit and Jenkis](https://www.hiroom2.com/2016/07/28/ubuntu-16-04-gerrit-and-jenkins-for-code-review)<br>
:link: [Replication configuration file](https://gerrit.googlesource.com/plugins/replication/+doc/master/src/main/resources/Documentation/config.md)<br>
:link: [Gerrit plugins](https://gerrit-review.googlesource.com/Documentation/config-plugins.html)<br>
:link: [Remove the Verified category](https://gerrit-review.googlesource.com/Documentation/prolog-cookbook.html#_example_9_remove_the_code_verified_code_category)<br>
:link: [Removing Verification label from a particular Gerrit project](https://groups.google.com/forum/#!topic/repo-discuss/fRN7YrcR6gw)<br>
:link: [config-labels](https://gerrit-review.googlesource.com/Documentation/config-labels.html#label_Verified)<br>
:link: [Test SSH Gerrit connection](https://review.openstack.org/Documentation/user-upload.html#test_ssh)<br>
:link: [Invalid command proxy requests when setting up jenkins](https://stackoverflow.com/questions/15850465/invalid-command-proxyrequests-when-setting-up-jenkins)<br>
:link: [Maven proxies](http://maven.apache.org/guides/mini/guide-proxies.html)<br>
:link: [Gerrit plugins](https://stackoverflow.com/questions/27177793/gerrit-and-github-com-googlesource-gerrit-plugins-github-oauth-oauthfilter)<br>
:link: [Permission denied- publickey](https://gerrit-review.googlesource.com/Documentation/error-permission-denied.html)<br>
</p></details>


[create_repo_github]: ./images/create_repo_github.png
[create_repo_gerrit]: ./images/create_repo_gerrit.png
[create_repo_gerrit_b]: ./images/create_repo_gerrit_b.png
[new_group]: ./images/new_group.png
[project_permissions_a]: ./images/project_permissions_a.png
[project_permissions_b]: ./images/project_permissions_b.png
[project_permissions_c]: ./images/project_permissions_c.png
[project_permissions_d]: ./images/project_permissions_d.png
[project_access]: ./images/project_access.png
[gerrit_hub]: ./images/gerrithub.jpg
[jenkins_gerrit]: https://gerrit-ci.gerritforge.com
[gerrit_releases]: https://gerrit-releases.storage.googleapis.com/index.html
[maven_env]: https://www.tutorialspoint.com/maven/maven_environment_setup.htm
[maven_proxy]: https://stackoverflow.com/questions/1251192/how-do-i-use-maven-through-a-proxy