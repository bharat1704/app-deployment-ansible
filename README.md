# Deploying a Static Web Application with Nginx Using Ansible

## Overview
This guide explains how to deploy a static web application using **Nginx** and **Ansible**. It covers:
- Setting up the correct **root directory**
- Ensuring **permissions**
- Configuring **Nginx** for static sites
- Handling **Single Page Applications (SPA) routing**

## Steps to Set Up Nginx for Static Hosting

### 1️⃣ Install Nginx
To install Nginx on a Linux system:
```yaml
- name: Install Nginx
  apt:
    name: nginx
    state: present
    update_cache: yes
```
🔹 This ensures Nginx is installed and updated.

### 2️⃣ Create the Web Root Directory
```yaml
- name: Create web root directory
  file:
    path: /var/www/html/app
    state: directory
    owner: www-data
    group: www-data
    mode: '0755'
```
🔹 This ensures:
- The directory `/var/www/html/app` exists.
- `www-data` (Nginx user) has the correct permissions.

### 3️⃣ Copy Application Files
```yaml
- name: Copy application files
  copy:
    src: ../app/
    dest: /var/www/html/app/
    owner: www-data
    group: www-data
    mode: '0644'
```
🔹 This copies your static site files (HTML, CSS, JS) to `/var/www/html/app/`.

### 4️⃣ Copy Nginx Configuration
```yaml
- name: Copy Nginx configuration
  copy:
    src: nginx.conf
    dest: /etc/nginx/sites-available/app
    owner: root
    group: root
    mode: '0644'
```
🔹 Places the Nginx config file in `/etc/nginx/sites-available/app`.

### 5️⃣ Enable the Site (Create a Symlink)
```yaml
- name: Enable site
  file:
    src: /etc/nginx/sites-available/app
    dest: /etc/nginx/sites-enabled/app
    state: link
```
🔹 Creates a **symbolic link** to enable the site.

### 6️⃣ Restart Nginx
```yaml
- name: Restart Nginx
  service:
    name: nginx
    state: restarted
```
🔹 Applies changes by restarting Nginx.

---

## Nginx Configuration for Static Files
Make sure the **root** directive in `nginx.conf` matches the correct directory:
```nginx
server {
    listen 80;
    server_name 134.199.218.137;
    #enter the public ip of your VM 

    root /var/www/html/app; # 👈 Must match your deployment folder
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```
✅ Ensures static files are correctly served.

---

## Handling SPA Routing (React, Vue, Angular)
SPAs use client-side routing, so Nginx needs to always serve `index.html`:
```nginx
server {
    listen 80;

    root /var/www/html/app;
    index index.html;

    location / {
        try_files $uri /index.html; # 👈 Ensures React/Vue handles routing
    }
}
```
✅ This ensures routes like `example.com/about` load correctly.

---

## Summary
| Step | Action |
|------|--------|
| **1** | Install Nginx |
| **2** | Create web root directory |
| **3** | Copy static files |
| **4** | Copy Nginx config |
| **5** | Enable site (symlink) |
| **6** | Restart Nginx |
| **7** | Configure root path in `nginx.conf` |
| **8** | Add SPA routing if needed |

✅ **Following these steps ensures your static site or SPA is served correctly with Nginx.**

Deployment Process:

Run the Ansible playbook:

ansible-playbook -i inventory.ini ansible/deploy.yaml
 
Ensure the correct static file is served.