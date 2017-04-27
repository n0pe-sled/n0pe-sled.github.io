---
layout: post
title:  "Mod_Rewrite Automatic Setup"
date:   2017-04-17 19:45:31 +0530
categories: Archive
author: Julian Catrambone
tags: [Mod-Rewrite, Phishing, Server-Setup]
---

Setting up infrastructure for a Red Team engagement can be time-consuming and difficult.  [Jeff Dimmock](https://twitter.com/bluscreenofjeff) and [Steve Borosh](https://twitter.com/424f424f) have done a lot of work to make this process easier and more transparent.  They gave a [great presentation](https://speakerdeck.com/rvrsh3ll/doomsday-preppers-fortifying-your-red-team-infrastructure) that went over the fundamentals of setting up good Red Team infrastructure, as part of this effort they released a [wiki](https://github.com/bluscreenofjeff/Red-Team-Infrastructure-Wiki).

One of the most interesting bits of trade craft released in this talk and on [Jeff's blog](https://bluescreenofjeff.com/tags#mod_rewrite) is their very creative use of apache2’s mod_rewrite functionality. Mod_Rewrite is very powerful for a few reasons:

1. Mod_Rewrite proxy connections hide the true location of your team server.
2. Mod_Rewrite user-agent redirects can be used to redirect mobile users away from a payload to a spoofed login portal.
3. Block specific IP addresses from your team server.
4. Only allow Malleable C2 traffic to the team server.

In a Red Team engagement, there are often multiple team servers, and multiple redirectors in front of each team server. In the event that a defender identifies and blocks one of the redirectors, they should be easy to recreate. However, manually setting up mod_rewrite rule set for each redirector can be very difficult and time-consuming. To make this easier, I automated the setup process and tried to include as much functionality as possible.

I will go over initial server configuration done by the script, and a few different use cases and how to quickly implement them.

## Server Initialization:

In order to initialize mod_rewrite we must tell Apache to allow a .htaccess file to override rules in the apache2 configuration file(/etc/apache2/apache2.conf).  In the Apache2 configuration file the default web-root is “/var/www/html/”.  You can see this pictured below.

![config1]({{ site.url }}/assets/Mod-Rewrite/Apache2-Config.PNG)

To allow the use of a .htaccess file, “AllowOverride None” needs to be changed to “AllowOverride All”.

Finally, in order to use mod_rewrite we need to enable a few apache2 modules. To enable these modules manually, run:

a2enmod rewrite proxy proxy_http

The script will take care of all of this for you.  Each time the script is run it will check that mod_rewrite is enabled and configured correctly. By default, it will enable and configure the default web-root of “/var/www/” but you can specify which web-root you would like to use with the --server_root flag.

![config]({{ site.url }}/assets/Mod-Rewrite/config.png)

## 1) Only allowing Malleable C2 traffic to the team server:

In order to accomplish this copy the script and the Malleable C2 profile on the server you are running the redirector on.

{% highlight bash %}
git clone https://github.com/n0pe-sled/Apache2-Mod-Rewrite-Setup.git
git clone https://github.com/rsmudge/Malleable-C2-Profiles.git
{% endhighlight %}

After you have a local copy, run the script with the following parameters:

{% highlight bash %}
python apache_redirector_setup.py --malleable="<Path to C2 Profile>" --block_url="https://google.com" --block_mode="redirect" --allow_url="team server Address" --allow_mode="proxy"
{% endhighlight %}

It will process the profile and create a set of rules that implement redirection based on the C2 Profile you provided.  These rules are written to “/var/www/html/” by default, but you can specify a different apache2 web-root with the --server_root flag.

![c2]({{ site.url }}/assets/Mod-Rewrite/C2.png)


## 2) Redirecting Mobile Users:

Mobile users can be identified by the user agent on the request. This script will set up user agent redirection for the following user agents when the --mobile_url, and --mobile_mode flags are used: android, blackberry, googlebot-mobile, iemobile, ipad, iphone, ipod, opera mobile, palmos, and webos.

If you would like to restrict any additional users they can be specified with the --block_ua flag. Here is a sample command that will setup mobile user agent redirection:

{% highlight  bash%}
python apache_redirector_setup.py --mobile_url="<mobile site>" --block_url="https://google.com" --block_mode="redirect" --allow_url="<team server Address>" --allow_mode="proxy"  --block_ua="<any additional ua to block>"
{% endhighlight %}

## 3) Blocking Specific IP Ranges or Addresses,

This script can set up redirection based on an IP Range or an IP Address. However, its functionality is limited to single ipv4 addresses, or /8,16,24 sub nets.

For example to block a single IP address:

{% highlight bash %}
python apache_redirector_setup.py --ip_blacklist="1.1.1.1;1.1.1.2" --block_url="https://google.com" --block_mode="redirect" --allow_url="<Team server Address>" --allow_mode="proxy
{% endhighlight %}

This will block the IP Addresses 1.1.1.1 and 1.1.1.2.  Here is an example on how to block IP ranges:
{% highlight bash %}
python apache_redirector_setup.py --ip_blacklist="1.1.1;1.2;3" --block_url="https://google.com" --block_mode="redirect" --allow_url="<Team server Address>" --allow_mode="proxy
{% endhighlight %}

This will block the 1.1.1.0/24 range, 1.2.0.0/16 range, and 3.0.0.0/8 range.

## 4) Complex Rulesets

All of these flags can be used together except when using malleable C2. For example, let's say that we would like a redirector that did mobile user redirection, blocked common IR user agents, blocked a /24 network, and only allowed specific request URI’s to reach the team server. The script will take each parameter and process them in this order:

1: IR Blacklisting
2: IP Blacklisting
3: UA Blacklisting
4: URI Whitelisting
5: Mobile Proxy/Redirect
6: Allow Clause

{% highlight bash %}
python apache_redirector_setup.py --mobile_url="https://mobile-payload.com" --mobile_mode="proxy" --valid_uris="payload;uploads" --ir --ip_blacklist="1.1.1" --block_url="https://GetBlockedNerd.com" --block_mode="redirect" --allow_url="https://Teamserver.com" --allow_mode="proxy"
{% endhighlight %}

![combo]({{ site.url }}/assets/Mod-Rewrite/Combo.png)

## Conclusion

Mod_Rewrite is very powerful, and can be as simple or complicated as you make it. This script is not intended to do everything that mod_rewrite has to offer and it never will. But, this serves as a good baseline and introduction to the power of mod_rewrite. The script can be found on my [Github](https://github.com/n0pe-sled/Apache2-Mod-Rewrite-Setup).

Huge shout out to [Jeff Dimmock](https://twitter.com/bluscreenofjeff) for guiding me through the process, and being there for me. I also have to give a shout out to all the current and former ATD members for teaching me almost everything I know about Red Teaming.


