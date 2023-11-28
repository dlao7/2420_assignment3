# Assignment 3 Part 1

## Connect to your new server as root

Where to find the host IP address:
Under server information, listed next to ipv4

```
ssh -i <path to ssh key> root@<host-ip-address>
```

The server will ask if you want to continue connecting when connecting for the first time. Type yes when prompted.

## Create a New User 

**Create** a new user, setting bash as their login shell and giving them access to elevated privileges with sudo

```bash
useradd -ms /bin/bash -G sudo <username>
```

**Set** the user's password.

```bash
passwd <username>
```

**Copy** the .ssh folder from the root folder to the new user's home directory

```bash
cp -r /.ssh /home/<username>
```

**Change** the user and group ownership of the .ssh folder in the new user's home directory to be owned by the user and user's primary group

```bash
chown -R <username>:<username> /home/<username>/.ssh
```

**Log out** as root user
```powershell
exit
```

**Connect** as the new user
```powershell
ssh -i <path to ssh key> <username>@<host-ip-address>
```
The server will ask if you want to continue connecting when connecting for the first time. Type yes when prompted.

### Remove Root Access to Server

**Rationale**: The root user has the highest level of privileges and can change anything on the server. Malicious hackers will attempt to login as root to access your server as it is the default user, and removing root login access increases your server security.

**Open** the sshd_config file in the /etc folder in the vim editor.

```bash
sudo vim /etc/ssh/sshd_config
```
In vim:
Search for a line in the document "PermitRootLogin yes"
and change this to "PermitRootLogin no"

**Save** the file and exit.

**Restart** the ssh service to save the changes.
```bash
sudo systemctl restart ssh.service
```

**Log out** from the server.
```powershell
exit
```

**Confirm** that you cannot connect to server as root.

```powershell
ssh -i <path to ssh key> root@<host-ip-address>

Should give error message:
root@<host-ip-address>: Permission denied (publickey).
```

## Nginx Installation and Configuration

### Install nginx

```bash
sudo apt update
sudo apt install nginx
```

### Configure nginx to serve a sample website

**Check** if nginx server is running
```bash
sudo systemctl status nginx
```
Under active, should state it is **active (running)**.

###  Create a sample html file
The /var/www/ directory contains root directories with html that can be served.

In the /var/www/ directory, create a new directory called my-site and create an index.html file inside that directory.

Sample code for index.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>2420</title>
    <style>
        body {
            display: flex;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
        }
        h1 {
            text-align: center;
        } 
    </style>
</head>
<body>
    <h1>Hello, World</h1>
</body>
</html>
```

The directory /etc/nginx/ contains nginx configuration files.

In /etc/nginx, the directory sites-available contains configuration files for the server. There is a default file which is currently set to serve a default file index.nginx-debian.html.

In vim, create a new file called my-site.conf and save it with the following code.

```vim
server {
  listen 80;
  listen [::]:80;
  
  root /var/www/my-site;
  
  index index.html index.htm;
  
  server_name _;
  
  location / {
  # First attempt to serve request as file, then
  # as directory, then fall back to displaying a 404.
  try_files $uri $uri/ =404;
  }
}
```
To **enable** a site to be served by the web server, you should add a symbolic link in sites-enabled to the configuration file in sites-available.

For my-site.conf,
```bash
sudo ln -s /etc/nginx/sites-available/my-site.conf /etc/nginx/sites-enabled/my-site
```
Note that you should use the absolute path for both target and destination directory to ensure that the link points to the correct location, as the ln utility will create symbolic links even if the target does not exist.

Currently, any requests to port 80 are set to serve files from both the default and my-site root directories, which is not possible. Therefore, you need to **remove** the symbolic link in sites-enabled to the default file.

```bash
sudo unlink /etc/nginx/sites-enabled/default
```

**Test** your nginx configuration:

```bash
sudo nginx -t

Success message:
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

**Restart** the nginx service
```bash
sudo systemctl restart nginx
```

**Open** a web browser and go to the server's ip address (the host ip address) and the html file saved in my-site should **display** in the browser if you successfully configured the nginx server.










