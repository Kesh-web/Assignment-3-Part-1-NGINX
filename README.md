# Assignment-3-Part-1-NGINX

## Introduction

This assignment focuses on building and configuring a simple automated system to generate and serve system information as a static HTML page. The task involves scripting, system administration, and secure server management. Specifically, we will:

- Create a system user for secure script execution.[1]
- Automate script execution daily using `systemd` services and timers.
- Configure Nginx to serve the HTML file on an Arch Linux server.
- Secure the server using a firewall configured with `ufw`.

The assignment includes using skills such as Bash scripting, `systemd` management, web server configuration, and basic firewall security.


## Table of contents

## Task 1: Setting up System User with Ownership and Directories 

[1]: The first step is creating the system user `webgen` with a home directory of `/var/lib/webgen`, giving it ownership, and  an appropriate login shell for a non-login user.

   ```bash
   sudo useradd -r -d /var/lib/webgen -s /usr/sbin/nologin webgen
```
### Explanation of `useradd` Options

- **`-r` system account:**  
  Creates a system user with a UID below 1000. System users are typically used for non-login tasks like running services.

- **`-d` home directory:**  
  Specifies the user’s home directory path, in our case its  `/var/lib/webgen`.

- **`-s` shell:**  
  Sets the login shell. Using `/usr/sbin/nologin` disables interactive logins for `webgen`, which is ideal for system users.

<br>

The benefit of using these specific options when creating the system user is

**Non-login Tasks:**  
   System users are typically used for non-login tasks, such as running services, by ensuring that they do not require interactive login capabilities.

### Creating Subdirectories `/bin` and `/HTML`

We need to create the following subdirectories within the `webgen` directory. To do so type the following:

```bash
sudo mkdir -p /var/lib/webgen/bin
sudo mkdir -p /var/lib/webgen/HTML
```

### Cloning the `generate_index` Script into `/bin/`

To set up the `generate_index` script, you can clone it from the sourcehut repo and place it in the `/bin/` directory. 


1. First cd into the `/bin/` directory:
```bash
cd /var/lib/webgen/bin
```

2. Second clone the repo with the following 
```bash 
git clone https://sr.ht/~user/repo.git generate_index
```
3. Third make sure the script is executable
```bash 
sudo chmod +x /var/lib/webgen/bin/generate_index
```

### Setting Ownership for the `webgen` System User

By running the following command `webgen` user has ownership of its home directory and all subdirectories and files.
```bash
sudo chown -R webgen:webgen /var/lib/webgen
```

### Explanation of Setting Ownership Options

`-R`
 command changes the ownership of `/var/lib/webgen` and all files and subdirectories within it, making sure that all files in the directory are owned by the `webgen` user and group.




## Task 2:
To start the first step would be creating the service file `generate-index.service` that would run the `generate_index` script.

```bash 
sudo nvim /etc/systemd/system/generate-index.service
```
Once youre in there you can edit it and add the following:

```ini
[Unit]
Description=Generate Index Service File

[Service]
Type=simple
User=webgen
Group=webgen
ExecStart=/var/lib/webgen/bin/generate_index
```
<br>
<hr/>

### Creating `generate-index.timer` script
Next we will create a timer that runs the script daily at 5:00pm; to do so write the following command.

```bash
sudo nano /etc/systemd/system/generate-index.timer
```
Once youre in the script you can add the following into the init file.

```ini
[Unit]
Description=Runs the generate_index script daily at 5:00pm 

[Timer]
OnCalendar=*-*-* 05:00:00
Unit=generate-index.service
Persistent=true

[Install]
WantedBy=timers.target
```



### Explanation of the Service File

#### [Unit]:
- **Description**: A brief description of the service.

#### [Service]:
- **Type=simple**: This tells systemd that the service is a simple service that does not fork. It starts immediately and runs in the foreground. 
- **ExecStart=/var/lib/webgen/bin/generate_index**: Specifies the command that will be executed when the service starts. Here, it runs the `generate_index` script located in `/var/lib/webgen/bin/generate_index`.

- **Type=simple**: This means that the service is considered started immediately when the `ExecStart` command 
<hr/>

<br>

After creating the service file and the ini file we need to start and enable one of them so it runs everytime you boot up your computer.

```bash
sudo systemctl start generate-index.timer
sudo systemctl enable generate-index.timer
sudo systemctl start generate-index.service
``` 

To check and see if the timer is active and service is run you can run the following commands.

```bash
systemctl status generate-index.timer
systemctl status generate-index.service
```



![status_timer_active_+_enabled](images/status-timer.png)

As well as using the journalctl command to see the logs of `generate-index.service`.

```bash
journalctl -u generate-index.service
```
This would be the output showing that it will succesfully be run daily at 5:00 pm.


![journalctl_service](images/journalctl_service.png)

(Renemmber to add missing picture)x



## Task 3:
To start we would need to modify the main `nginx.conf` file to ensure the server runs as the `webgen` user. Before we do that we have to download nginx

```bash
sudo pacman -Syu nginx
```
After downloading `nginx` we can then start to edit the main file with the folllowing command.

```bash
  sudo nvim /etc/nginx/nginx.conf
  ```

You want to change the user from the default to `
  webgen` and it should look like the following image.

![nginx-user](images/nginx-user.png)

### Adding a Server Block

A server block in Nginx defines how requests are processed for a specific domain or IP address. It allows hosting multiple websites on a single server with different configurations. We will edit it using the following command.

```bash
sudo nvim /etc/nginx/nginx.conf
```



```nginx
server {
    listen 80;
    server_name _;

    root /var/lib/webgen/HTML/index;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### Explanation of the Server Block

- **`listen 80;`**  
  Configures Nginx to listen on port 80 for HTTP traffic.

- **`server_name _;`**  
  The `_` acts as a wildcard, making this server block respond to all requests if no other server block matches.

- **`root /var/lib/webgen/HTML/index;`**  
  Sets the root directory for the website files.

- **`index index.html;`**  
  Specifies the default file to serve (e.g., `index.html`) when a directory is requested.

- **`location / { try_files $uri $uri/ =404; }`**  
  Ensures Nginx tries to serve the requested file or directory. If it doesn't exist, a `404 Not Found` error is returned.










## References Page

https://wiki.archlinux.org/title/Users_and_groups