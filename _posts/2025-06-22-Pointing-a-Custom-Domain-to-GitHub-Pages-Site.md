---
layout: post
title: "Pointing a custom domain to a GitHub Pages hosted site"
draft: false
date:  2025-06-22
description: How to point a personal domain to your site the you host on GitHub pages.  Easy setup.  
---
 
## **Pointing a Custom Domain to a GitHub Pages Site**

**Situation:**  
 You own a domain (e.g., example.com) via a registrar like Namecheap, and your site is hosted on GitHub Pages (e.g., yourusername.github.io).

**Steps:**

1. Login to your domain registrar (e.g., Namecheap)

2. Go to Domain List \> Manage \> Advanced DNS

3. Delete any default A or CNAME records

4. Add A Records for GitHub Pages:

   * Host: @

   * Values (add one A record for each IP):

     * 185.199.108.153

     * 185.199.109.153

     * 185.199.110.153

     * 185.199.111.153

   * TTL: Automatic

5. Add a CNAME Record:

   * Host: www

   * Value: yourusername.github.io

   * TTL: Automatic

6. In your GitHub repo \> Settings \> Pages:

   * Set Custom Domain to: example.com

   * GitHub will automatically enable HTTPS

7. Add a CNAME file to your repo (if not already):

   * Filename: CNAME

   * Contents: example.com

   * Commit and push

8. Wait 15–60 minutes for DNS to propagate

9. Visit your domain and check HTTPS and redirects

## **Pointing a Domain to a Site Hosted on Your Own Web Server**

**Situation:**  
 You own a domain and run your own web server (e.g., on a home server or cloud VPS).

**Steps:**

1. Find your public IP address of the web server

2. Login to your domain registrar

3. Go to Domain List \> Manage \> Advanced DNS

4. Delete any existing A records

5. Add A Record:

   * Host: @

   * Value: your server's IP address (e.g., 123.45.67.89)

   * TTL: Automatic

6. Optional: Add CNAME for www

   * Host: www

   * Value: example.com

   * TTL: Automatic

7. Ensure your server is configured to serve content for that domain

   * Update Apache, NGINX, or similar config

8. Set up HTTPS with Let's Encrypt (optional but recommended)

   * Use Certbot or similar

9. Wait a few minutes and test the domain

## **Pointing a Domain to a Budget Web Host (e.g., Bluehost, HostGator)**

**Situation:**  
 You own a domain and want it to point to a site hosted on a shared Linux web host.

**Steps:**

1. Login to your web host account

   * Find the DNS settings, IP address, or nameservers in your hosting dashboard

2. Login to your domain registrar

3. Option A: Use Nameservers (recommended by hosts)

   * Replace your registrar's default nameservers with those provided by the host

   * Example:

     * ns1.hostgator.com

     * ns2.hostgator.com

   * Wait for propagation (up to 24 hours)

**OR**

4. Option B: Use A Record (manual setup)

   * Host: @

   * Value: your host's IP address

   * TTL: Automatic

   * Optional: Add CNAME

     * Host: www

     * Value: example.com

5. Configure your site within the hosting control panel

   * Set the domain to point to the right directory

6. Wait for DNS to propagate

7. Visit your domain and verify everything works

