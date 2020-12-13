# Rundeck

## Install server

```text
docker run \
>   -tid \
>   -p 4440:4440 \
>   --name rundeck \
>   jordan/rundeck:latest
```

### References

[https://hub.docker.com/r/jordan/rundeck](https://hub.docker.com/r/jordan/rundeck)

## Tips and Tricks

### Install plugin

Download the plugin jar file and put it here:

```text
/var/lib/rundeck/libext/
```

### Add node

Change into the newly created project directory with the command:

```text
cd /var/rundeck/projects/PROJECTNAME/etc/
```

Where PROJECTNAME is the name of the project you just created.

Create a new configuration file with the command:

```text
nano resources.xml
```

The contents of the file will look like this:

```text
<?xml version="1.0" encoding="UTF-8"?>
<project>
<node name="NAME"
  osFamily="unix"
  username="USERNAME"
  hostname="REMOTE_IP"
  ssh-authentication="password"
  sudo-command-enabled="true"
  sudo-password-storage-path="keys/USERNAME"
  />
</project>
```

Where:

* NAME is the name of the remote node.
* USERNAME is the username on the remote node.
* REMOTE\_IP is the IP address of the remote node.

Save and close that file.

Restart Rundeck with the command:

```text
sudo systemctl restart rundeckd
```

#### References

[https://www.techrepublic.com/article/how-to-add-remote-nodes-to-rundeck/](https://www.techrepublic.com/article/how-to-add-remote-nodes-to-rundeck/)

[https://stackoverflow.com/questions/36224912/add-a-remote-node-in-rundeck](https://stackoverflow.com/questions/36224912/add-a-remote-node-in-rundeck)

