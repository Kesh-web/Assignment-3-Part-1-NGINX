# Assignment-3-Part-1-NGINX

## Introduction

This assignment focuses on building and configuring a simple automated system to generate and serve system information as a static HTML page. The task involves scripting, system administration, and secure server management. Specifically, we will:

- Create a system user for secure script execution.
- Automate script execution daily using `systemd` services and timers.
- Configure Nginx to serve the HTML file on an Arch Linux server.
- Secure the server using a firewall configured with `ufw`.

The assignment includes using skills such as Bash scripting, `systemd` management, web server configuration, and basic firewall security.

>[!NOTE]
> Ensure you have root or sudo privileges to perform the tasks outlined in the following.

## Table of contents
- [Introduction](#introduction)
- [Table of contents](#table-of-contents)
- [Task 1: Setting up System User with Ownership and Directories](#task-1-setting-up-system-user-with-ownership-and-directories)
  - [Explanation of `useradd` Options](#explanation-of-useradd-options)
  - [Creating Subdirectories `/bin` and `/HTML`](#creating-subdirectories-bin-and-html)
  - [Cloning the `generate_index` Script into `/bin/`](#cloning-the-generate_index-script-into-bin)
  - [Setting Ownership for the `webgen` System User](#setting-ownership-for-the-webgen-system-user)
  - [Explanation of Setting Ownership Options](#explanation-of-setting-ownership-options)
- [Task 2: Creating and Configuring `systemd` Service and Timer](#task-2-creating-and-configuring-systemd-service-and-timer)
  - [Creating the Service File](#creating-the-service-file)
  - [Creating the Timer Script](#creating-the-timer-script)
  - [Explanation of the Service File](#explanation-of-the-service-file)
  - [Starting and Enabling the Timer and Service](#starting-and-enabling-the-timer-and-service)
- [Task 3: Configuring Nginx](#task-3-configuring-nginx)
  - [Adding a Server Block in a New Configuration File](#adding-a-server-block-in-a-new-configuration-file)
  - [Explanation of the Server Block](#explanation-of-the-server-block)
  - [Checking Nginx Service Status](#checking-nginx-service-status)
- [Task 4: Installing and Configuring `ufw`](#task-4-installing-and-configuring-ufw)
  - [Fixing iptables Error](#fixing-iptables-error)
  - [Enabling Rate Limiting](#enabling-rate-limiting)
  - [Allowing HTTP Traffic](#allowing-http-traffic)
- [Task 5: Verifying the Setup](#task-5-verifying-the-setup)
- [Conclusion](#conclusion)
- [References](#references)

## Task 1: Setting up System User with Ownership and Directories 

The first step is creating the system user `webgen` with a home directory of `/var/lib/webgen`, giving it ownership, and an appropriate login shell for a non-login user.

  ```bash
  sudo useradd -r -d /var/lib/webgen -s /usr/sbin/nologin webgen
  ```

### Explanation of `useradd` Options

- **`-r` system account:**  
  Creates a system user with a UID below 1000. System users are typically used for non-login tasks like running services.

- **`-d` home directory:**  
  Specifies the user’s home directory path, in our case its `/var/lib/webgen`.

- **`-s` shell:**  
  Sets the login shell. Using `/usr/sbin/nologin` disables interactive logins for `webgen`, which is ideal for system users.

>[!IMPORTANT]
> Using these specific options ensures that the `webgen` user is configured securely for non-login tasks.

### Creating Subdirectories `/bin` and `/HTML`

We need to create the following subdirectories within the `webgen` directory. To do so, type the following:

```bash
sudo mkdir -p /var/lib/webgen/bin
sudo mkdir -p /var/lib/webgen/HTML
```

### Cloning the `generate_index` Script into `/bin/`

To set up the `generate_index` script, you can clone it from the sourcehut repo and place it in the `/bin/` directory. 

1. First, change into the `/bin/` directory:
  ```bash
  cd /var/lib/webgen/bin
  ```

2. Second, clone the repo with the following:
  ```bash 
  git clone https://git.sr.ht/~nathan_climbs/2420-as2-start generate_index
  ```

3. Third, make sure the script is executable:
  ```bash 
  sudo chmod +x /var/lib/webgen/bin/generate_index
  ```

### Setting Ownership for the `webgen` System User

By running the following command, the `webgen` user will have ownership of its home directory and all subdirectories and files:
```bash
sudo chown -R webgen:webgen /var/lib/webgen
```

### Explanation of Setting Ownership Options

- **`-R` recursive:**  
  This command changes the ownership of `/var/lib/webgen` and all files and subdirectories within it, ensuring that all files in the directory are owned by the `webgen` user and group.

>[!CAUTION]
> Ensure you use the `-R` option to avoid permission issues with subdirectories and files.

## Task 2: Creating and Configuring `systemd` Service and Timer

### Creating the Service File

The first step is creating the service file `generate-index.service` that will run the `generate_index` script.

```bash 
sudo nvim /etc/systemd/system/generate-index.service
```

Once you're in the editor, add the following:

```ini
[Unit]
Description=Generate Index Service File
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=webgen
Group=webgen
ExecStart=/var/lib/webgen/bin/generate_index
```

### Explanation of the Service File

- **Description:** A brief description of the service.
- **After=network-online.target:** Ensures the service starts only after the network is online.
- **Wants=network-online.target:** Indicates that the service wants the network to be online, but it is not a strict dependency.

After creating the service file, reload the systemd manager configuration to apply the changes:

```bash
sudo systemctl daemon-reload
```


### Creating the Timer Script

Next, we will create a timer that runs the script daily at 5:00 AM. To do so, write the following command:

```bash
sudo nano /etc/systemd/system/generate-index.timer
```

Once you're in the editor, add the following:

```ini
[Unit]
Description=Runs the generate_index script daily at 5:00 AM

[Timer]
OnCalendar=*-*-* 5:00:00
Unit=generate-index.service
Persistent=true

[Install]
WantedBy=timers.target
```

### Explanation of the Service File

#### [Unit]
- **Description:** A brief description of the service.

#### [Service]
- **Type=simple:** This tells systemd that the service is a simple service that does not fork. It starts immediately and runs in the foreground.
- **ExecStart=/var/lib/webgen/bin/generate_index:** Specifies the command that will be executed when the service starts. Here, it runs the `generate_index` script located in `/var/lib/webgen/bin/generate_index`.

>[!NOTE]
> The `Type=simple` option means that the service is considered started immediately when the `ExecStart` command is executed.

### Starting and Enabling the Timer and Service

After creating the service file and the timer file, we need to start and enable the timer so it runs every time you boot up your computer.

```bash
sudo systemctl start generate-index.timer
sudo systemctl enable generate-index.timer
```

To check and see if the timer is active and the service is running, you can run the following commands:

```bash
systemctl status generate-index.timer
systemctl status generate-index.service
```

![status_timer_active_+_enabled](images/status-timer.png)

You can also use the `journalctl` command to see the logs of `generate-index.service`.

```bash
journalctl -u generate-index.service
```

This would be the output showing that it will successfully run daily at 5:00 PM.

![journalctl_service](images/journalctl_service.png)


## Task 3: Configuring Nginx

To start, we need to modify the main `nginx.conf` file to ensure the server runs as the `webgen` user. Before we do that, we have to download Nginx.

```bash
sudo pacman -Syu nginx
```

After downloading Nginx, we can then start to edit the main file with the following command:

```bash
sudo nvim /etc/nginx/nginx.conf
```

You want to change the user from the default to `webgen` as well as uncomment it if it is commented out and it should look like the following image:


![nginx-user](images/nginx-user.png)

### Adding a Server Block in a New Configuration File

Instead of modifying the main `nginx.conf` file, we will create a new directory for custom configuration files and add a server block in a new configuration file.

1. **Create a new directory for custom configuration files:**

  ```bash
  sudo mkdir -p /etc/nginx/sites-available
  sudo mkdir -p /etc/nginx/sites-enabled
  ```

2. **Create a new configuration file for the server block:**

  ```bash
  sudo nvim /etc/nginx/sites-available/webgen.conf
  ```

3. **Add the following server block configuration to the new file:**

  ```nginx
  server {
     listen 80;
     server_name local-webgen;

     root /var/lib/webgen/HTML;
     index index.html;

     location / {
        try_files $uri $uri/ =404;
     }
  }
  ```

4. **Create a symbolic link to enable the new configuration:**

  ```bash
  sudo ln -s /etc/nginx/sites-available/webgen.conf /etc/nginx/sites-enabled/
  ```

5. **Include the new directory in the main `nginx.conf` file:**

  ```bash
  sudo nvim /etc/nginx/nginx.conf
  ```

  Add the following line inside the `http` block:

  ```nginx
  include /etc/nginx/sites-enabled/*.conf;
  ```

### Explanation of the Server Block

- **`listen 80;`**  
  Configures Nginx to listen on port 80.

- **`server_name local-webgen;`**  
  Defines the server name.

- **`root /var/lib/webgen/HTML;`**  
  Sets the root directory for the files.

- **`index index.html;`**  
  Specifies the default file to serve `index.html` when a directory is requested.

- **`location / { try_files $uri $uri/ =404; }`**  
  Ensures Nginx tries to serve the requested file or directory. If it doesn't exist, a `404 Not Found` error is returned.

>[!IMPORTANT]
> Using a separate server block file instead of modifying the main `nginx.conf` file directly helps maintain an organized configuration. It allows you to make updates and changes to an individual server without affecting the main `nginx.conf`, and if there are any issues, you can isolate the problem more easily.

### Checking Nginx Service Status

To make sure that Nginx is running properly and the config file we made works, we can use the following commands:

1. **Check the status of the Nginx service:**

  ```bash
  sudo systemctl status nginx
  ```

2. **Test the Nginx configuration for syntax errors:**

  ```bash
  sudo nginx -t
  ```

  This command will check the Nginx config files for any syntax errors.

3. **Restart Nginx to apply any changes:**

  ```bash
  sudo systemctl restart nginx
  ```
### Explanation of the Server Block

- **`listen 80;`**  
  Configures Nginx to listen on port 80 for HTTP traffic.

- **`server_name local-webgen;`**  
  Defines the server name for the block.

- **`root /var/lib/webgen/HTML;`**  
  Sets the root directory for the website files.

- **`index index.html;`**  
  Specifies the default file to serve (e.g., `index.html`) when a directory is requested.

- **`location / { try_files $uri $uri/ =404; }`**  
  Ensures Nginx tries to serve the requested file or directory. If it doesn't exist, a `404 Not Found` error is returned.

>[!IMPORTANT] 
> Using a separate server block file instead of modifying the main `nginx.conf` file directly helps maintain an organized configuration. It allows you to make updates and changes to an individual server without affecting the main `nginx.conf`, and if there are any issues, you can isolate the problem more easily.g

### Checking Nginx Service Status

To make sure that Nginx is running properly and the config file we made works, we can use the following commands:

1. **Check the status of the Nginx service:**

  ```bash
  sudo systemctl status nginx
  ```

2. **Test the Nginx configuration for syntax errors:**

  ```bash
  sudo nginx -t
  ```

  This command will check the Nginx config files for any syntax errors.

3. **Restart Nginx to apply any changes:**

  ```bash
  sudo systemctl restart nginx
  ```

  ## Task 4: Installing and Configuring `ufw`
To start we will first install `ufw`, we can do so by typing in the following command.
```bash
sudo pacman -Syu ufw
```

>[!CAUTION]
> Do not enable `ufw` immediately after installation, as it may lock you out of your SSH session. Ensure you have allowed SSH connections before enabling the firewall.

```bash
sudo ufw allow ssh
```
To enhance security, we can enable SSH rate limiting to prevent brute-force attacks. Use the following command to set up it up.

```bash
sudo ufw limit ssh
```



>[!NOTE]
> This setting helps to mitigate brute-force attacks by limiting the rate of SSH login attempts.

```bash
sudo ufw allow http
```
This command allows HTTP traffic through the firewall, enabling web traffic to reach your Nginx server.

>[!NOTE]
> Allowing HTTP traffic is essential for serving web pages to users.


### Fixing iptables Error

If you encounter an iptables error after allowing a rule, follow these steps to resolve it. If you do not encounter this error, you can skip to the next section on enabling rate limiting.

Possible error message:

![ip-tables](images/ip-tables.png)

#### Steps to Fix iptables Error:

1. **Update your system:**

  ```bash
  sudo pacman -Syu
  ```

2. **Update the outdated iptables version:**

  ```bash
  sudo pacman -S iptables
  ```

3. **Restart iptables:**

  ```bash
  sudo systemctl restart iptables
  ```

  After following these commands it will fix the problem.

### Enabling Rate Limiting

To enhance security, we can enable SSH rate limiting to prevent brute-force attacks. Use the following command to set it up:

```bash
sudo ufw limit ssh
```

This setting helps to mitigate brute-force attacks by limiting the rate of SSH login attempts.

### Allowing HTTP Traffic

Allow HTTP traffic through the firewall to enable web traffic to reach your Nginx server:

```bash
sudo ufw allow http
```

Allowing HTTP traffic is essential for serving web pages to users.



This will display the current status and rules of the firewall, including the SSH rate limiting and HTTP allowance.


After configuring the rate limiting, you can enable `ufw`:

```bash
sudo ufw enable
```

To check the status of `ufw` and confirm that the rules are applied, use the following command.

```bash
sudo ufw status
```
![ufw-status](images/ufw-status.png)


This will display the current status and rules of the firewall, including the SSH rate limiting.

## Task 5: Verifying the Setup

With everything configured, visit your droplet's IP address in the browser and take a screenshot of the system information page. The screenshot should clearly show your IP address and the system information page.

![system-information](images/system-information.png)

## Conclusion

Congrats! You have successfully set up your server to generate and serve system information as a static HTML page!!!


## References

- [Arch Linux Wiki: Users and Groups](https://wiki.archlinux.org/title/Users_and_groups)
- [Arch Linux Wiki: Example Adding a System User](https://wiki.archlinux.org/title/Users_and_groups#Example_adding_a_system_user)
- [Arch Linux Wiki: Nginx](https://wiki.archlinux.org/title/Nginx)
- [Arch Linux Wiki: Nginx Server Blocks](https://wiki.archlinux.org/title/Nginx#Server_blocks)
- [Arch Linux Wiki: Running Nginx as a Specific User](https://wiki.archlinux.org/title/Nginx#Running_as_a_specific_user)
- [Arch Linux Wiki: Systemd](https://wiki.archlinux.org/title/Systemd)
- [Arch Linux Wiki: Running services after the network is up](https://wiki.archlinux.org/title/Systemd#Running_services_after_the_network_is_up)
- [Arch Linux Wiki: Systemd Timers](https://wiki.archlinux.org/title/Systemd/Timers)
- [Arch Linux Wiki: Uncomplicated Firewall](https://wiki.archlinux.org/title/Uncomplicated_Firewall)
- [Arch Linux Wiki: Rate Limiting with Ufw](https://wiki.archlinux.org/title/Uncomplicated_Firewall#Rate_limiting_with_ufw)

