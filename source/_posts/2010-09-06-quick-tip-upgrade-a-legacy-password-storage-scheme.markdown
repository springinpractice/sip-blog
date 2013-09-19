---
layout: post
title: "Quick tip: upgrade a legacy password storage scheme"
date: 2010-09-06 22:25:39
comments: true
categories: [Chapter 06 - Authentication, Quick Tips]
---
This one's a Spring Security quick tip that I wanted to share. Suppose that you have a password storage scheme that stores passwords as plaintext, and you want to upgrade that to storing hashes. No problem; simply replace the plaintext passwords with hashed versions (e.g., SHA-256).

But what if your legacy scheme is to store hashed passwords, and you want to upgrade that to store salted, hashed passwords? Here you don't have the original passwords, so you can't construct salted, hashed versions of the originals, at least not without resorting to reversing the passwords, which we generally don't want to do. How do we proceed?

It's easy. Just use multiple authentication providers pointing at the same source: one for hashed passwords, and one for salted, hashed passwords. Here's how it looks in Spring Security 3:
<pre style="margin:20px 0;">&lt;authentication-manager&gt;

    &lt;!-- For legacy passwords --&gt;
    &lt;authentication-provider user-service-ref="userDetailsService" /&gt;

    &lt;!-- Salted, hashed passwords --&gt;
    &lt;authentication-provider user-service-ref="userDetailsService"&gt;
        &lt;password-encoder ref="passwordEncoder"&gt;
            &lt;salt-source ref="saltSource" /&gt;
        &lt;/password-encoder&gt;
    &lt;/authentication-provider&gt;
&lt;/authentication-manager&gt;

</pre>

The first provider handles the legacy simple hash-based scheme, and the second handles the new scheme.
