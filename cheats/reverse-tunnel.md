---
description: >-
  Open SSH connections to servers under NAT without needing to redirect ports on
  the router.
---

# Reverse tunnel

Consider the following scenario: you are about to deploy a server on-premisse and you cannot deal with router ports redirection and firewall rules.

The solution is open a reverse tunnel, so you have a secure backdoor open to SSH connections:

## Setup the tunnel server

This server \(which can be on the cloud\) will handle all connections from on-premisse servers.

### Install SSH server

```bash
apt update
apt install openssh-server
```

### Add an user called `tunnel`

```bash
useradd -m -s /bin/bash tunnel
```

### Setup the RSA key

Login as `tunnel` user.

```bash
su - tunnel
```

Create an RSA key.

```bash
ssh-keygen -t rsa -b 4096
```

{% hint style="info" %}
Accept all defaults \(**do not** input a password\)
{% endhint %}

Now authorize tunnel's user RSA public key in its authorized\_keys.

{% hint style="info" %}
Remember we are creating a reverse tunnel. So the on-premisse server will use the same private RSA key, so the compatible public key must be authorized.
{% endhint %}

Copy the RSA public key.

```bash
cat /home/tunnel/.ssh/id_rsa.pub
```

Now edit `/home/tunnel/.ssh/authorized_keys`

{% code title="/home/tunnel/.ssh/authorized\_keys" %}
```bash
nano /home/tunnel/.ssh/authorized_keys
```
{% endcode %}

Add the following content.

{% code title="/home/tunnel/.ssh/authorized\_keys" %}
```text
command="",restrict,port-forwarding PUT-THE-PUBLIC-KEY-HERE
```
{% endcode %}

It will look like this.

{% code title="/home/tunnel/.ssh/authorized\_keys" %}
```text
command="",restrict,port-forwarding ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQChoOyIa8Z5VtXm0AKYxsszkjwtfI7h4I00rgMQOKoWF6DR14l43fQrNq3I/lpUJBCOxHwcH6QkPCI/Oj8oXQkyg7NgpDBQ6y7IDaP5RX3MCUTcrjfvjUyuc3NPS3loB5g6bHFpkyULj9mmQp4/IOfCL55bzenQQSUQmmDk4LGAFQVjiBoIYFMEkpar87QYX/qktP2np+z3jpUD6Dz2F5MVccxbqsoHJZmSPBtw1rV9cr61HebPvrK5Ir6r50PXOVVRHbH0lILJdxOQOxbslDkcq5SypsEYSNsiAArWwzxM4YcXaTt/vzL8/fqsy8Nhx26BjmGTQ8Yptsl2KX7VdpF95Q7fHL7ElVr6C+COb2BOoEls/pHgCJzEbjKSnLPKTFacovPjzbch52MqCBNWsrr9JRHuxLP+35VgD5LqhVlGkeOuqKVx+TQcsp4ITKBpKX6d2iFni26SynjnzsLAdqOGSrYy7ZTRlHQnJGrY9LsbUrxVrM6+tZZq2PiCdbxhtsg1IKxp14lAF/L6ZOgpUxJlgZNIepxUfBYEEdwGIXHOhUv4/4ipmgMPCOVKlvTwNaGURW7RNHkn2iziYd7EUcWpKdkLK4IQMg9UXmbxg7HyhvZGcLBeLpmBnUBiN7+NJGbn/DLultYLcNIUN90FHgdJ+r0RE91FKsqW0UI0FLXJIw== tunnel@myhost.com
```
{% endcode %}

{% hint style="info" %}
`command="",restrict,port-forwarding` will restrict the access. If someone get physical access to the on-premisse server it will not be possible to allocate a TTY using the RSA key. 
{% endhint %}

Copy the content of the following files \(we will use them to set up the on-premisse server\).

```bash
cat /home/tunnel/.ssh/id_rsa
cat /home/tunnel/.ssh/id_rsa.pub
```

## Setup the on-premisse server

This server will open the reverse tunnel to the tunnel server.

### Install SSH server

```bash
apt update
apt install openssh-server
```

### Setup the RSA key

Let's suppose you want the user ubuntu to be able to open the tunnel.

Create the RSA key.

```bash
nano /home/ubuntu/.ssh/tunnel.id_rsa
```

Paste the content from `/home/tunnel/.ssh/id_rsa` you copied from the tunnel server.

Fix permissions.

```bash
chmod 700 /home/ubuntu/.ssh/tunnel.id_rsa
```

Add tunnel server's public RSA to authorized keys.

```bash
nano /home/ubuntu/.ssh/authorized_keys
```

Paste the content from `/home/tunnel/.ssh/id_rsa.pub` you copied from the tunnel server.

## Test connection

### From your on-premisse server

Open a tunnel to test the connection.

```bash
/usr/bin/ssh \
  -o ServerAliveInterval=20 \
  -o ServerAliveCountMax=3 \
  -o ExitOnForwardFailure=yes \
  -o StrictHostKeyChecking=no \
  -i /home/ubuntu/.ssh/tunnel.id_rsa \
  -N -T -R1822:localhost:22 tunnel@PUT-YOUR-TUNNEL-SERVER-HOST-HERE
```

{% hint style="warning" %}
Replace `PUT-YOUR-TUNNEL-SERVER-HOST-HERE` with your tunnel server host or IP.
{% endhint %}

{% hint style="info" %}
You can change the port 1822 to any port you want \(just keep me same port for the rest of the tutorial\)
{% endhint %}

This command will open a the tunnel using the **port** **1822** on the **tunnel server**. All connections on port 1822 on the tunnel server will be redirected to **port 22** on the **on-premisse server**.

If everything is OK the command above will hang the tunnel. Leave it running.

### From your tunnel server

Login as tunnel user and check if the tunnel port is open.

```bash
netstat -natp |grep 1822
```

You should see something like.

```bash
tcp        0      0 127.0.0.1:1822         0.0.0.0:*               LISTEN      -
```

Now test the tunnel.

```bash
ssh -o StrictHostKeyChecking=no ubuntu@localhost -p 1822
```

You should be able to SSH to your on-premisse server.

## Configure the tunnel as a service on the on-premisse server

Go back to your on-premisse server. You can cancel/stop the test tunnel we opened before.

Create the following file.

```bash
sudo nano /etc/systemd/system/ssh-tunnel.service
```

With the following content.

{% code title="/etc/systemd/system/ssh-tunnel.service" %}
```text
[Unit]
Description=Create a tunnel in the cloud back to SSH on this machine
After=network-online.target

[Service]
User=ubuntu
ExecStart=/usr/bin/ssh -o ServerAliveInterval=20 -o ServerAliveCountMax=3 -o ExitOnForwardFailure=yes -o StrictHostKeyChecking=no -i /home/ubuntu/.ssh/tunnel.id_rsa -N -T -R1822:localhost:22 tunnel@PUT-YOUR-TUNNEL-SERVER-HOST-HERE
RestartSec=3
Restart=always

[Install]
WantedBy=multi-user.target
```
{% endcode %}

{% hint style="warning" %}
Replace `PUT-YOUR-TUNNEL-SERVER-HOST-HERE` with your tunnel server host or IP.
{% endhint %}

{% hint style="info" %}
You can change the port 1822 to any port you want
{% endhint %}

Then start and enable the service to start on boot.

```bash
sudo systemctl daemon-reload
sudo systemctl enable ssh-tunnel.service
sudo systemctl start ssh-tunnel.service
```

