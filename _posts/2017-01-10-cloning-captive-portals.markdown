---
layout: post
title:  "Cloning and Hosting Evil Captive Portals using a Wifi PineApple"
date:   2017-01-10 19:45:31 +0530
categories: Archive
author: Julian Catrambone
---
Recently, I was on a Wifi Assessment and one of our objectives was to obtain the Access Code to a guest wireless network.  To do this we decided to use a [Wifi Pineapple - Tetra](https://wifipineapple.com/).

For this post I will go over the process of cloning a website to use for your captive portal using the [Portal Auth](https://github.com/sud0nick/PortalAuth) module, then hosting it with the [Evil Portal](https://github.com/frozenjava/evilportal) Module.  I will not be going over the initial setup of the Wifi Pineapple.  For more information on setup I suggest you look [here](https://www.youtube.com/watch?v=gqMW0NeODAQ).

So lets get started!  The first thing we need to do is download the "Portal Auth" and "Evil Portal" modules.
![Modules]({{ site.url }}/assets/Captive-Portals/modules.png)

Click install and then click internal storage to install the module
![Install-snap]({{ site.url }}/assets/Captive-Portals/Install-snap.png)

After they are both installed we can start to setup Portal Auth!  Portal Auth is going to help us capture and create a captive portal to host.  First, click "use default" to populate all of the configuration options.
![portal_auth_setup.png]({{ site.url }}/assets/Captive-Portals/portal_auth_setup.png)

Next we have to pick a website to clone for the portal.  For this example I am going to use https://www.starbucks.com/.  Under "Test Site" put the url of the site you would like to clone.  Then click "save".  If the Pineapple can find and connect to the Test Site the Clone Portal Button should appear. :)
![portal_auth_setup2.png]({{ site.url }}/assets/Captive-Portals/portal_auth_setup-2.png)

Now that we have a webpage set we need to make some slight changes to an injection set.  Under Injection Sets click "Edit Injects" and select "Harvester"
![portal_auth_setup3.png]({{ site.url }}/assets/Captive-Portals/portal_auth_setup-3.png)

Here we will edit the HTML for the situation.  In my case I was looking for an Access Code and Name.  Below is my modified HTML:

{% highlight ruby %}
<div id="pa_overlay-back"></div>
<div id="pa_msgBox" class="pa_main">
	<h1 class="pa_h1">Internet access is on us today.</h1><br />
	<h4 class="pa_h4">Please enter your Name and Access Code to gain access.</h4>
	<br /><br />
	<div>
		<input type="text" id="pa_email" name="pa_email" class="pa_field" placeholder="Name" />
	</div>
	<br />
	<div>
		<input type="password" id="pa_password" name="pa_password" class="pa_field" placeholder="Access Code" />
	</div>
	<br /><br />
	<button id="submit_button" class="pa_connectButton" type="button">Connect</button>
</div>
{% endhighlight %}

Once you have made your changes make sure to click "Save HTML" to overwrite the previous injection.

Now all that is left is to click "Clone Portal" and set a Portal Name, and under Injection Set select Harvester.
![portal_auth_setup4.png]({{ site.url }}/assets/Captive-Portals/portal_auth_setup-4.png)

Once it has finished cloning we have to use the "Evil Portal" Plugin to host the newly created captive portal.  Navigate to the Evil Portal Module, and Activate the captive portal by pressing "Start" under the controls panel.  Once Started Activate your saved portal.  When the portal is activated and the captive portal is started you can view a preview of the portal under live preview!
![evil_portal_setup.png]({{ site.url }}/assets/Captive-Portals/evil_portal_setup.png)

Finally all we have to do is set up the public AP for our target to connect too!  Under "Networking" set the "Open AP SSID" to something that fits your target, ensure that "Hide Open AP" is not checked, then Click "Update Access Point".  Now your captive portal should be up and running!
![network-setup.png]({{ site.url }}/assets/Captive-Portals/networking-setup.png)

You can test it by connecting to your malicious SSID and using the captive portal.  Here is our example running!
![network-setup.png]({{ site.url }}/assets/Captive-Portals/Final.png)
