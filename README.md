<!---
{
  "id": "f81d1db6-ad8e-42ff-916c-157049d29292",
  "teaches": "Serving Local Directories with Apache and Virtual Hosts",
  "depends_on": ["b9e34253-61e2-4fb7-bd36-388c370fa765"],
  "author": "Stephan Bökelmann",
  "first_used": "2025-05-06",
  "keywords": ["apache", "localhost", "virtualhosts", "subdomains", "webserver"]
}
--->

# Serving Local Directories with Apache and Virtual Hosts

> In this exercise you will learn how to configure the Apache HTTP Server to serve local directories on `127.0.0.1`. Furthermore we will explore how to use name-based virtual hosts to simulate subdomains locally using your system's hosts file and Apache's configuration directives.

### Introduction

Apache HTTP Server (often referred to simply as "Apache") is a powerful, flexible, and widely-used open-source web server software. It allows developers to serve web content from their local machines, making it essential for testing, debugging, and developing web applications. Setting up Apache to serve content locally is one of the foundational skills for any web developer or DevOps engineer.

In a local setup, Apache can be configured to bind only to the loopback interface (`127.0.0.1`), effectively making the server accessible only from the local machine. This is useful for development and testing purposes, especially when working on web applications that are not yet ready for public deployment. Additionally, using virtual hosts (vhosts), you can simulate multiple websites or subdomains on your local environment. This allows you to test configurations like `project.local`, `api.project.local`, or any other subdomain structure you might eventually use in production.

This exercise will guide you through the full setup: installing Apache, setting it to bind to `127.0.0.1`, serving a custom directory, and finally, using vhosts to serve different subdomains. Make sure you have administrator (root) access on your machine, as modifying system files like `/etc/hosts` and restarting Apache services will require elevated privileges.

### Further Readings and Other Sources

* [Official Apache HTTP Server Documentation](https://httpd.apache.org/docs/)
* [DigitalOcean Guide: How to Set Up Apache Virtual Hosts](https://www.digitalocean.com/community/tutorials/how-to-set-up-apache-virtual-hosts-on-ubuntu-20-04)
* [YouTube: Apache Virtual Hosts Tutorial](https://www.youtube.com/watch?v=BS7H4v5ZLoE)

---

## Tasks

### Task 1: Install Apache HTTP Server

1. Begin by updating your package lists and upgrading existing packages to ensure compatibility:

   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

2. Install the Apache HTTP server package:

   ```bash
   sudo apt install apache2
   ```

3. Verify the installation:

   ```bash
   apache2 -v
   systemctl status apache2
   ```

### Task 2: Bind Apache to 127.0.0.1 Only

1. Edit the ports configuration:

   ```bash
   sudo nano /etc/apache2/ports.conf
   ```

2. Modify the `Listen` directive:

   ```
   Listen 127.0.0.1:80
   ```

3. Ensure no conflicting processes on port 80:

   ```bash
   sudo lsof -i :80
   ```

4. Restart Apache:

   ```bash
   sudo systemctl restart apache2
   ```

### Task 3: Serve a Local Directory

1. Create a project directory:

   ```bash
   mkdir -p ~/websites/project1
   echo "<h1>Hello from Project1</h1>" > ~/websites/project1/index.html
   ```

2. Create a virtual host file:

   ```bash
   sudo nano /etc/apache2/sites-available/project1.conf
   ```

3. Use and understand this configuration:

   ```apache
   <VirtualHost 127.0.0.1:80>
       ServerName project1.local
       DocumentRoot /home/YOUR_USERNAME/websites/project1

       <Directory /home/YOUR_USERNAME/websites/project1>
           Options Indexes FollowSymLinks
           AllowOverride None
           Require all granted
       </Directory>
   </VirtualHost>
   ```

   * `<VirtualHost 127.0.0.1:80>`: Applies to traffic on loopback IP and port 80.
   * `ServerName`: Defines the domain name Apache responds to.
   * `DocumentRoot`: The folder served to users visiting this domain.
   * `<Directory>`: Applies access rules to that folder.
   * `Options Indexes FollowSymLinks`: Allows directory listing and following symlinks.
   * `AllowOverride None`: Disables `.htaccess` overrides.
   * `Require all granted`: Permits all local access.

4. Enable the site:

   ```bash
   sudo a2ensite project1.conf
   sudo systemctl reload apache2
   ```

5. Add to `/etc/hosts`:

   ```bash
   sudo nano /etc/hosts
   ```

   Add:

   ```
   127.0.0.1   project1.local
   ```

6. Visit [http://project1.local](http://project1.local) in your browser.

### Task 4: Serve Subdomains with Virtual Hosts

1. Create subdomain directory:

   ```bash
   mkdir -p ~/websites/api.project1
   echo "<h1>Hello from API</h1>" > ~/websites/api.project1/index.html
   ```

2. Create subdomain vhost file:

   ```bash
   sudo nano /etc/apache2/sites-available/api.project1.conf
   ```

3. Paste and understand this:

   ```apache
   <VirtualHost 127.0.0.1:80>
       ServerName api.project1.local
       DocumentRoot /home/YOUR_USERNAME/websites/api.project1

       <Directory /home/YOUR_USERNAME/websites/api.project1>
           Options Indexes FollowSymLinks
           AllowOverride None
           Require all granted
       </Directory>
   </VirtualHost>
   ```

   * Like before, this serves content only on 127.0.0.1 for the subdomain `api.project1.local`.

4. Enable site and reload:

   ```bash
   sudo a2ensite api.project1.conf
   echo "127.0.0.1 api.project1.local" | sudo tee -a /etc/hosts
   sudo systemctl reload apache2
   ```

5. Open [http://api.project1.local](http://api.project1.local) in a browser.

---

## Advice

If something doesn't work right away, don't panic. Debugging configuration issues is a big part of working with local servers. Always check Apache's error log (`/var/log/apache2/error.log`) when things go wrong. Be meticulous with file paths and make sure permissions allow the Apache process to read your directories. Working with virtual hosts is a powerful way to mirror production environments locally, so mastering this setup will serve you well in both development and operations roles. Once you're comfortable, you can build on this by adding SSL via self-signed certificates or exploring how similar configurations work in Nginx. Consider reviewing the [advanced Apache configuration sheet](#) once you've completed this one.
