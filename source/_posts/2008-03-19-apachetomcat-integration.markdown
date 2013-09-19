---
layout: post
title: "Apache/Tomcat integration"
date: 2008-03-19 13:51:54
comments: true
categories: [Tutorials]
---
Here's a step-by-step tutorial that shows how to integrate Tomcat 5.5.x with Apache 2.0.x under Windows XP using the AJP 1.3 connector. The exact versions I used for this tutorial were Tomcat 5.5.9 and Apache 2.0.54, but the instructions ought to work for Tomcat 5.5.x and Apache 2.0.x generally.

<h3>Preliminaries</h3>

<h4>Step 1</h4>

Decide which directory you want to use to deploy your web app. In a bit we will be pointing both Tomcat and Apache at this directory.

<h3>Configure Tomcat</h3>

<h4>Step 2</h4>

In your Tomcat <code>server.xml</code> file, make sure that your AJP 1.3 connector config is not commented out. Mine looks like this:

<pre>&lt;Connector port="8009" 
    enableLookups="false"
    redirectPort="8443"
    protocol="AJP/1.3"/&gt;</pre>

and it immediately follows the config for the HTTP connector running on port 8080 (Tomcat default). The AJP connector listens on port 8009 for requests that Apache forwards to Tomcat. The idea is that Apache services the requests for static content itself, and forwards requests requiring servlet/JSP processing. Incidentally, if you don't intend to use Tomcat as a standalone server, you should comment out the Coyote standalone connector that listens on port 8080.

<h4>Step 3</h4>

Also in <code>server.xml</code>, point the Catalina engine at the directory you just chose. In my own case, the config looks like this:

<pre>&lt;Engine name="Catalina" defaultHost="localhost"&gt;
    &lt;Host name="localhost" appBase="C:/cygwin/home/web/deploy"
          unpackWARs="true" autoDeploy="true"
          xmlValidation="false" xmlNamespaceAware="false"&gt;
         &lt;Valve className="org.apache.catalina.valves.AccessLogValve"
                directory="C:/cygwin/home/web/logs" prefix="localhost_access_log."
                suffix=".txt" pattern="common" resolveHosts="false"/&gt;
    &lt;/Host&gt;
&lt;/Engine&gt;</pre>

<h3>Configure Apache</h3>

<h4>Step 4</h4>

Download the <code>mod_jk</code> binary from the URL given in the References section below. The specific version I am using is a Windows binary, <code>mod_jk-1.2.14-apache-2.0.54.so</code>. <code>mod_jk</code> is the Apache module that allows Apache to talk to Tomcat.

<h4>Step 5</h4>

Copy the file you just downloaded into the Apache modules directory, but note that you need to rename it to <code>mod_jk.so</code>.

<h4>Step 6</h4>

Put a <code>workers.properties</code> file in the Apache configuration directory, right next to <code>httpd.conf</code>. Here is a sample <code>workers.properties</code> file; obviously you'll need to modify it to match your own local environment.

<h4>Step 7</h4>

Now modify <code>httpd.conf</code> to tell Apache about <code>mod_jk</code> and about our <code>workers.properties</code> file. Note that the worker name you choose here needs to match the worker name that you defined in <code>workers.properties</code>. Also, you will have to decide which requests get forwarded to Tomcat, and specify that here as JK mountpoints. Here's the configuration I'm using; modify as necessary:

<pre># Load mod_jk module
# Update this path to match your modules location
LoadModule jk_module C:/apache-2.0.54/Apache2/modules/mod_jk.so

# Where to find workers.properties
# Update this path to match your conf directory location
# (put workers.properties next to httpd.conf)
JkWorkersFile C:/apache-2.0.54/Apache2/conf/workers.properties

# Where to put jk logs
# Update this path to match your logs directory location
# (put mod_jk.log next to access_log)
JkLogFile C:/cygwin/home/web/logs/mod_jk.log

# Set the jk log level [debug/error/info]
JkLogLevel info

# Select the log format
JkLogStampFormat "[%a %b %d %H:%M:%S %Y] "

# JkOptions indicate to send SSL KEY SIZE, 
JkOptions +ForwardKeySize +ForwardURICompat -ForwardDirectories

# JkRequestLogFormat set the request format 
JkRequestLogFormat "%w %V %T"

# Specify which requests get forwarded to Tomcat.  (My web app is called
# "mywebapp"; change as necessary.  Note that I'm forwarding Struts *.do
# requests to Tomcat, along with the top-level home page request, which in my
# case happens to be a Struts request.)
JkMount /mywebapp/ ajp13w
JkMount /mywebapp/*.do ajp13w
JkMount /mywebapp/*.jsp ajp13w</pre>

<h4>Step 8</h4>

Modify <code>httpd.conf</code> so that it obtains documents from the same directory you chose in step 1 above.

That should do it. If you access your web app on port 80, things should hopefully work the way you want them to. Important: do not try to access your web app on port 8080, especially if you commented out the Coyote connector in <code>server.xml</code>. The whole point of the above was to put Tomcat behind Apache, and now we have that. Client requests come in on port 80, and if there are requests whose URIs match the JkMount URIs specified in <code>httpd.conf</code>, then Apache forwards those to Tomcat on port 8009 via an AJP 1.3 request.

<h3>References</h3>
<ul>
<li><a href="http://tomcat.apache.org/tomcat-5.5-doc/config/ajp.html">The AJP Connector</a>: Explains what you need to do in <code>server.xml</code> if you want to get fancy with your AJP config.</li>
</ul>

<div class="endnote">Post migrated from my Wheeler Software site.</div>
