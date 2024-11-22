# Assignment-3-Part-1-NGINX

## Introduction

This assignment focuses on building and configuring a simple automated system to generate and serve system information as a static HTML page. The task involves scripting, system administration, and secure server management. Specifically, we will:

- Create a system user for secure script execution.
- Automate script execution daily using `systemd` services and timers.
- Configure Nginx to serve the HTML file on an Arch Linux server.
- Secure the server using a firewall configured with `ufw`.

The assignment emphasizes critical skills such as Bash scripting, `systemd` management, web server configuration, and basic firewall security, ensuring a well-rounded understanding of server administration.


## Table of contents

## Task 1: Setting up System User with Ownership and Directories

The first step is creating the system user `webgen` with a home directory of `/var/lib/webgen`, giving it ownership, and  an appropriate login shell for a non-login user.

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

### Setting Ownership for the `webgen` System User

By running the following command `webgen` user has ownership of its home directory and all subdirectories and files.
```bash
sudo chown -R webgen:webgen /var/lib/webgen
```


## Task 2:


### Explanation of Setting Ownership Options

`-R`
 command changes the ownership of `/var/lib/webgen` and all files and subdirectories within it, making sure that all files in the directory are owned by the `webgen` user and group.



















## References Page

https://wiki.archlinux.org/title/Users_and_groups