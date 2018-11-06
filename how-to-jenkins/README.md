# Table of contents

- [Create jenkins account in Gerrit Code Review](#create-jenkins-account-in-gerrit-code-review)
- [Install Java](#install-java)
- [Disable Password Authentication in SSH](#disable-password-authentication-in-ssh)
- [Disable Guest Account (ubuntu 16.04)](#disable-guest-account--ubuntu-1604-)
- [Installing jenkins](#installing-jenkins)
- [Setup nginx (reverse proxy)](#setup-nginx--reverse-proxy-)
  * [Settings for HTTPS](#settings-for-https)
  * [Configure Jenkins to use the reverse proxy (nginx)](#configure-jenkins-to-use-the-reverse-proxy--nginx-)
  * [Restart Jenkins](#restart-jenkins)
  * [Download missing plugins (if any)](#download-missing-plugins--if-any-)
  * [Run jenkins with trusted certificates from gerrit with HTTPS](#run-jenkins-with-trusted-certificates-from-gerrit-with-https)
    + [1. Import certificate](#1-import-certificate)
    + [2. Duplicate Java Keystore file and move into Jenkins ...](#2-duplicate-java-keystore-file-and-move-into-jenkins-)
    + [3. Add Certificate to Keystore](#3-add-certificate-to-keystore)
    + [4. Add -Djavax.net.ssl.trustStore=$JENKINS_HOME/keystore/cacerts to the](#4-add--djavaxnetssltruststore--jenkins-home-keystore-cacerts-to-the)
    + [Restart jenkins in order to apply the changes](#restart-jenkins-in-order-to-apply-the-changes)



## Create jenkins account in Gerrit Code Review

```bash
cat id_rsa.pub | ssh -p 29418 hiperezr@starlingxcodereview.intel.com gerrit \
create-account --ssh-key - jenkins --group "'Non-Interactive Users'"
```

> Also add the `jenkins` user to the group `Event Streaming Users`

Reference: [cmd-create-account](
https://review.openstack.org/Documentation/cmd-create-account.html)

## Install Java

Install Java on the jenkins server and the nodes (if any)

```bash
# apt-get update
# apt-get install default-jre
# apt-get install openjdk-8-jdk
```

## Disable Password Authentication in SSH

:warning: Before this please add your `id_rsa.pub` to authorized_keys file into
`/home/$USER/.ssh` folder

```bash
# vim /etc/ssh/sshd_config
PasswordAuthentication no
PubkeyAuthentication yes
ChallengeResponseAuthentication no
```

Reference: [Initial Setup Server](
https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-16-04)

## Disable Guest Account (ubuntu 16.04)

```bash
# sh -c 'printf "[SeatDefaults]\nallow-guest=false\n" > \ 
/etc/lightdm/lightdm.conf.d/50-no-guest.conf'
```

Reference: [Remove guest session](
http://ubuntuhandbook.org/index.php/2016/04/remove-guest-session-ubuntu-16-04)

## Installing jenkins

Please refer to the following [documentation](
https://www.digitalocean.com/community/tutorials/how-to-install-jenkins-on-ubuntu-16-04)

## Setup nginx (reverse proxy)

```bash
# apt install nginx
# ufw allow 'Nginx HTTPS'
```

### Settings for HTTPS

Edit the file `/etc/nginx/sites-available/jenkins` with the following
information

```text
server {
        listen 80;
        server_name <SERVER_NAME|IP>;
        return 301 https://$host$request_uri;
}

server {
        listen 443;
        server_name <SERVER_NAME|IP>;

		ssl on;
		ssl_certificate conf.d/certificate.crt;
		ssl_certificate_key conf.d/certificate.key;
        location ^~ / {
                proxy_pass        http://127.0.0.1:8080;
				proxy_read_timeout  90;
				# The following lines fixes the Jenkins message:
				# - Jenkins says my reverse proxy setup is broken
				proxy_set_header X-Forwarded-Host $host:$server_port;
				proxy_set_header X-Forwarded-Server $host;
				proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
				proxy_set_header X-Forwarded-Proto $scheme;
				proxy_set_header X-Real-IP $remote_addr;
				proxy_redirect      http://<SERVER_NAME|IP>:8080 https://<SERVER_NAME|IP>;
        }
}
```

Do a soft link

```bash
# ln -s /etc/nginx/sites-available/jenkins /etc/nginx/sites-enabled
```

Open the required ports in the firewall

```bash
# ufw allow 80
# ufw allow 443
```

### Configure Jenkins to use the reverse proxy (nginx)

In the file `/etc/default/jenkins` locate the variable `JENKINS_ARGS` and add
`--httpListenAddress=127.0.0.1` to the existing arguments

### Restart Jenkins

```bash
# systemctl restart jenkins
# systemctl status jenkins
# systemctl restart nginx
# systemctl status nginx
```

Reference: [Jenkins with reverse proxy](
https://www.digitalocean.com/community/tutorials/how-to-configure-jenkins-with-ssl-using-an-nginx-reverse-proxy-on-ubuntu-18-04)

### Download missing plugins (if any)

```bash
cd var/lib/jenkins/plugins
wget http://updates.jenkins-ci.org/download/plugins/ace-editor/1.1/ace-editor.hpi
```

Restart jenkins

### Run jenkins with trusted certificates from gerrit with HTTPS

This avoid the following error using REST API from Gerrit (events-log) under
Gerrit Trigger Plugin Configuration

```text
Connection error : sun.security.validator.ValidatorException: PKIX path
building failed: sun.security.provider.certpath.SunCertPathBuilderException:
unable to find valid certification path to requested target
```

<details><summary>Click here to see the steps</summary><p>

```bash
JENKINS_HOME=/var/lib/jenkins
JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
$ALIAS="This can be any value. It is a value to distinguish this certificate
from others. Example would be “git-repo”, or “artifact server”."
```

#### 1. Import certificate

```bash
openssl s_client -showcerts -connect <hostname>:<443> < /dev/null 2> /dev/null | openssl x509 -outform PEM > ~/root_ca.pem
```

Note : `.pem` files is the same as `.crt` files, so if you have the `.crt`
file certificate please copy in jenkins server

#### 2. Duplicate Java Keystore file and move into Jenkins ...

```bash
mkdir -p $JENKINS_HOME/keystore/
# chown -R jenkins.jenkins $JENKINS_HOME/keystore
# cp $JAVA_HOME/jre/lib/security/cacerts $JENKINS_HOME/keystore/
```

#### 3. Add Certificate to Keystore

```bash
# keytool -import -alias $ALIAS -keystore $JENKINS_HOME/keystore/cacerts -file ~/root_ca.pem
Enter keystore password: The password of keystore by default is: "changeit"
...
...
Trust this certificate? [no]:  yes
Certificate was added to keystore
```

#### 4. Add -Djavax.net.ssl.trustStore=$JENKINS_HOME/keystore/cacerts to the
Jenkins startup parameters. For Debian/Ubuntu, this is /etc/default/jenkins

```bash
echo 'JAVA_ARGS="$JAVA_ARGS -Djavax.net.ssl.trustStore=$JENKINS_HOME/keystore/cacerts"' >> /etc/default/jenkins

# chown jenkins.jenkins /var/lib/jenkins/keystore/cacerts
```


#### Restart jenkins in order to apply the changes
```bash
# service jenkins restart
```

[reference 1](
https://stackoverflow.com/questions/24563694/jenkins-unable-to-find-valid-certification-path-to-requested-target-error-whil)<br>
[reference 2](
https://support.cloudbees.com/hc/en-us/articles/203821254-How-to-install-a-new-SSL-certificate-)<br>
[reference 3](
https://stackoverflow.com/questions/8640340/how-do-i-get-into-a-non-password-protected-java-keystore-or-change-the-password)<br>
</p></details>
