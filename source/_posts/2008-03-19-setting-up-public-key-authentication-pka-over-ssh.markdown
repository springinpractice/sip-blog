---
link: http://springinpractice.com/2008/03/19/setting-up-public-key-authentication-pka-over-ssh/
layout: post
title: Setting up public key authentication (PKA) over SSH
date: 2008-03-19 14:18:48
comments: true
categories: [Quick Tips]
---
I assume you already know the whys, concepts, and terminology; this is just a statement of the steps involved. I'm using OpenSSH and a DSA key pair.

<h3>Step 1</h3>

Generate a key pair:
	
<pre>ssh-keygen -t dsa
Generating public/private dsa key pair.
Enter file in which to save the key (/home/Willie/.ssh/id_dsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/Willie/.ssh/id_dsa.
Your public key has been saved in /home/Willie/.ssh/id_dsa.pub.
The key fingerprint is:
f2:f7:5b:9b:f7:64:2b:d8:fe:ca:ad:f5:13:35:9f:63 Willie@ARCATA</pre>

This creates your key pair, and places them in your <code>~/.ssh</code> directory. The public key is <code>id_dsa.pub</code>; the private key is <code>id_dsa</code>. If it isn't already obvious, the public key is not a secret, and the private key is. :-)
	
<h3>Step 2</h3>

Install the public key to any SSH servers for which you'd like to use PKA. To do this, SSH into the server machine and open up the <code>~/.ssh/authorized_keys</code> file in a text editor. If you haven't already installed a public key to the server in the past, then you'll be creating a new file. Just append the contents of your id_dsa.pub file to <code>authorized_keys</code>. Here's mine:
	
<pre>ssh-dss AAAAB3NzaC1kc3MAAACBAOybEZ4kAaKROXoibeR+V/ajTY3L/aN6K5lVbdWKsw+9uPl/cyj4
6Qu5UYHkLS5tiGci8Olx7jNfku4/k1z8/JoGDTqwAixMxgb/NNKTUB7ZnhxfVTenSI/oVtM/lNpCiOdg
U7ESOyNrxPFVU6K1pWId+LGxeweWTw+08vwIOShTAAAAFQDx6q5JWhV2EDGUMXFwj3QF8+8a4wAAAIBW
Mee5MphZPYxG7la772tAYREo+37eXfP3SW49GmPHJFdydFcf5VtroLlzKJ1Iy9HUwnKjiEv2qE1B2xVD
jJslgQ34QVKKswQDRCXlyshyKbbRMd37MSYNpNqdZ5gTJT+EMa8+NoTUGwXOitSMMtx2WmpVo4Fu7Fp1
eDYvSVChjAAAAIB6uisHso6iPMz11qbKNaHSIqIAV+7iNJZD7aeFuytLDG20Y70b4Jy4Mr4g8RH+MtAL
fyq6aTcv/g/j2DMeJjwjqLXQFbaFekmQEOfoQ6IZJ5CylthMP1PzRcR5KeCUInKj9CRkTlWLlTMk5es+
VEebIHg9SWNstkjWBLwlQhemgA== Willie@ARCATA</pre>

(For formatting purposes, I've included line breaks above, but don't introduce line breaks into your public key when you paste it into <code>authorized_keys</code>.)
	
<h3>Step 3</h3>

Adjust the permissions on your <code>.ssh</code> directory, and on the files inside it, so that nobody else can write to them.

<h3>Step 4</h3>

Set up the SSH agent. This allows you to enter your passphrase one time per shell session instead of having to type it in every time you want to SSH to your server. Just type <code>ssh-agent</code> followed by <code>ssh-add</code>. Some people just put this in their startup script (e.g., <code>.bash_profile</code>). You'll need to use <code>eval $(ssh-agent)</code> instead of <code>ssh-agent</code> if you choose to do that. You'll have to enter your passphrase every time you open a shell, but after that you can SSH anywhere where you've installed your public key without having to enter your passphrase.

<div class="endnote">Post migrated from my Wheeler Software site.</div>
