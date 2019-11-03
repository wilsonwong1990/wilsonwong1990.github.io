# Setting up Jenkins to run over https

While it is suggested to run [Jenkins https by putting a reverse proxy in front of it](https://wiki.jenkins.io/pages/viewpage.action?pageId=135468777), I didn't wanted to just run it straight off the VM, so I did the following:

## Generate SSL

```
openssl req \
       -newkey rsa:2048 -nodes -keyout jenkins.hostname.key \
       -x509 -days 3650 -out jenkins.hostname.crt
```
This generated a SSL cert that is good for 10 years. As this is just an internal homelab server, I didn't want to deal with renewing all the time. I'm still planning on potentially running a CA.

### Convert to jks and put in jenkins folder
```

openssl pkcs12 -inkey jenkins.hostname.key -in jenkins.hostname.crt -export -out keys.pkcs12

keytool -importkeystore -srckeystore keys.pkcs12 -srcstoretype pkcs12 -destkeystore /var/lib/jenkins/jenkins.jks

```

This will convert the SSL cert to jks and export it to `/var/lib/jenkins` where jenkins expects the file.


### Edit Jenkins file

file: /etc/sysconfig/jenkins
```
JENKINS_ARGS="--httpPort=-1 --httpsPort=8443 --httpsKeyStore=/var/lib/jenkins/jenkins.jks --httpsKeyStorePassword=passsword"
```

Add these arguments to your `/etc/sysconfig/jenkins` config to use port 8443 and enable https.

### Restart Jenkins
```
systemctl restart jenkins
```
Restart the service to use the new config.

### Network
```
sudo firewall-cmd --add-port=443/tcp --permanent
sudo firewall-cmd --add-forward-port=port=443:proto=tcp:toport=8443 --permanent
sudo firewall-cmd --reload
```
Set up firewall to open port 443 and forward the port to Jenkin's port 8443.
