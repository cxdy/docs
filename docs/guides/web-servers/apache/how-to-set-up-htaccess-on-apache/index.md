---
slug: how-to-set-up-htaccess-on-apache
author:
  name: Linode
  email: docs@linode.com
description: 'This in-depth guide on the .htaccess file covers how to handle permissions, redirects, and IP address restriction.'
keywords: ["htaccess", " apache"]
license: '[CC BY-ND 4.0](https://creativecommons.org/licenses/by-nd/4.0)'
published: 2017-09-25
modified: 2017-09-26
modified_by:
  name: Linode
title: 'How to Set Up the htaccess File on Apache'
contributor:
  name: Christopher Piccini
  link: https://twitter.com/chrispiccini11
external_resources:
- '[HTTP Error and Status Codes](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)'
- '[Apache .htaccess Documentation](https://httpd.apache.org/docs/current/howto/htaccess.html)'
tags: ["web server","apache"]
aliases: ['/web-servers/apache/how-to-set-up-htaccess-on-apache/']
---

![How to Set Up the htaccess File on Apache](how-to-set-up-the-htaccess-file-on-apache-smg.jpg)

## Introduction
The purpose of this guide is to show you how to set up htaccess configuration (.htaccess) for Apache. The guide will cover topics relating to handling website file structure permissions, redirects, and IP address restrictions.


## Before You Begin

1.  If you have not already done so, create a Linode account and Compute Instance. See our [Getting Started with Linode](/docs/guides/getting-started/) and [Creating a Compute Instance](/docs/guides/creating-a-compute-instance/) guides.

1.  Follow our [Setting Up and Securing a Compute Instance](/docs/guides/set-up-and-secure/) guide to update your system. You may also wish to set the timezone, configure your hostname, create a limited user account, and harden SSH access.

1.  Complete the Apache section in the [Install a Lamp Stack](/docs/guides/how-to-install-a-lamp-stack-on-ubuntu-18-04/) to install Apache on your Linode.

{{< note >}}
Throughout this guide, replace each instance of `testuser` with your custom user account. Replace each occurrence of `example.com` with the IP address or Fully Qualified Domain Name (FQDN) of your Linode.
{{< /note >}}

## What is .htaccess

.htaccess is a configuration file for the Apache web server. It's an extremely powerful tool that can be used to modify the Apache configuration without needing to edit the Apache configuration files. The following sections describe how to create this configuration and use it to restrict directory listings and IP addresses, and to handle redirects.

## Enable .htaccess

By default, .htaccess isn't available. To enable it you will need to edit the configuration file.

1. Use a text editor to open your configuration file:

        sudo nano /etc/apache2/sites-available/example.com.conf

2.  After the VirtualHost block (</VirtualHost>) add:

    {{< file "/etc/apache2/sites-available/example.com.conf" >}}
....
</VirtualHost>
<Directory /var/www/html/example.com/public_html>
    Options Indexes FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>

{{< /file >}}

3.  Save the file, then restart apache:

        sudo service apache2 restart


## Restrict Directory Listings

By default, someone visiting your website can view the directory and file structure, and gain access to files on the web server. It's best practice to restrict directory access, so that a visitor to example.com would have to be familiar with files on the server in order to see them. One way you can restrict this is through .htaccess.

### Create .htaccess
1.  CMS systems such as WordPress create .htaccess configurations by default. This guide assumes that no .htaccess file exists, so you will have to create one manually. Navigate to your site's root directory:

        cd /var/www/html/example.com/public_html

2.  Create an .htaccess file:

    {{< file "/var/www/html/example.com/public_html/.htaccess" >}}
Options -Indexes

{{< /file >}}


3.  Now if you navigate to your site, you will see a **Forbidden** message. You will have to specifically indicate the file or directory you would like to see.

## Restrict IPs

This section will guide you through restricting specific IPs from accessing your site. This is useful if you want to block certain visitors from visiting your site. You may also set this up to prevent certain IPs from accessing certain sections of your site.

{{< note >}}
Subdirectories can inherit settings from .htaccess files in their parent directories if they're not overridden by a separate .htaccess file in the subdirectory. The examples in this guide will continue to work with an .htaccess file in the project's root directory. You should carefully consider in which directory different .htaccess directives should be placed.
{{< /note >}}

### Block IPs

1.  Create or edit the .htaccess file located at the directory where Apache will host the website:

        cd /var/www/html/example.com/public_html/
        sudo nano .htaccess

2.  Delete the `Options -Indexes` line from the previous section (if applicable) and add the following lines to block the target IP addresses:

    {{< file "/var/www/html/example.com/public_html" >}}
order allow,deny

# This will deny the IP 192.0.2.9
deny from 192.0.2.9

# This will deny all IP's from 192.0.2.0 through 192.0.2.255
deny from 192.0.2

{{< /file >}}


### Allow IPs

1.  Create or edit the .htaccess file located in the web directory where you want this setting to be applied.

2.  Add the following lines to deny all IPs except for the specific IP and pool of IPs mentioned in the command:

    {{< file "/var/www/html/example.com/public_html" >}}
order deny,allow

# Denies all IP's
Deny from all

# This will allow the IP 192.0.2.9
allow from 192.0.2.9

# This will allow all IP's from 192.0.2.0 through 192.0.2.255
allow from 192.0.2

{{< /file >}}


## Handle Redirects

You can redirect traffic using .htaccess configuration. In the below example, you'll update the .htaccess file for the root directory of your website to redirect a visitor to `http://example.com/test1/index.html` if they try to visit `http://example.com/main.html`.

{{< note >}}
Ensure that your website has a landing page, in the following steps replace each instance of `main.html` with the landing page of your website.
{{< /note >}}

1.  Create a test html file to redirect a visitor to `http://example.com/test1/index.html`:

        mkdir test1
        sudo touch test1/index.html

2.  Add some basic content to the test html file:

    {{< file "/var/www/html/example.com/public_html/test1/index.html" >}}
<!doctype html>
<html>
  <body>
     This is the html file in test1.
  </body>
</html>

{{< /file >}}

3.  Open the .htaccess file in your project's root directory. Remove all existing configurations in this file and add the following line:

    {{< file "/var/www/html/example.com/public_html/.htaccess" >}}
Redirect 301 /main.html /test1/index.html

{{< /file >}}

The first parameter after the 'Redirect' command is the HTTP status code. Specifying a status code is helpful for letting the browser know that the page has been moved to a new location. If you leave this parameter blank, it defaults to a 302 code indicating that the redirect is temporary. Specifying 301 makes it clear that the page at the requested location has permanently moved to a new location.

The next parameter is the Unix path to the file that is requested in the URL. This parameter requires that it is a Unix path and not a URL. The path should be the location of the .htaccess file where the redirect configuration is set up. The final parameter indicates where you want the visitor to be redirected. In this case, the traffic is being redirected to `/test1/index.html`; for this second parameter a Unix path or HTTP URL is acceptable.

4.  Navigate to `example.com/main.html` in a browser. You should see the url redirect to `example.com/test1/index.html` in the address bar, and your test html file should be displayed.

###  Set the 404 Error Page

When a visitor attempts to access a page or resource that doesn't exist (for example by following a broken link or typing an incorrect URL,) the server will respond with a 404 error code. It is important that users receive feedback explaining the error. By default, Apache will display an error page in the event of a 404 error.  However, most sites provide a customized error page. You can use .htaccess settings to let Apache know what error page you would like displayed whenever a user attempts to access a nonexistent page.

1.  This will redirect all requests for nonexistent documents to a page in the project root directory called `404.html`.  Open the `.htaccess` file and add the following line:

    {{< file "/var/www/html/example.com/public_html/.htaccess" >}}
ErrorDocument 404 /404.html

{{< /file >}}


2.   Create the `404.html` file:

    {{< file "/var/www/html/example.com/public_html/404.html" >}}
<!doctype html>
<html>
 <body>
   404 Error: Page not found
 </body>
</html>

{{< /file >}}


3.  In a browser, navigate to a page that does not exist, such as `www.example.com/doesnotexist.html`. The 404 message should be displayed.
