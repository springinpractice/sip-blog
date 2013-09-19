---
link: http://springinpractice.com/2008/09/06/login-remember-me/
layout: post
title: Excerpt: Login and remember-me discussion
date: 2008-09-06 15:37:16
comments: true
categories: [Chapter 06 - Authentication, News]
---
[caption id="attachment_85" align="alignright" width="240" caption="A healthy human retina. Our book doesn&#039;t go into biometric authentication but this is a cool photo anyway."]<a href="http://www.flickr.com/photos/bike/2635107055/"><img class="size-full wp-image-85" title="Human retina" src="http://springinpractice.com/wp-content/uploads/2008/09/retina2.jpg" alt="" width="240" height="160" /></a>[/caption]

<em><a href="http://www.manning.com/wheeler/">Spring in Practice</a></em> centers on using Spring to implement technical solutions to common problems, but it's also important for developers to understand the problem they're trying to solve before implementing a solution. In the book we work pretty hard to provide that understanding.

Here's an except from the discussion section for a recipe on implementing login forms and remember-me authentication using Spring Security. While discussion sections might treat the problem, the solution or both, this particular discussion digs a little deeper some security/usability tradeoffs to consider when implementing username/password and remember-me authentication.
<blockquote>
<h3>Discussion</h3>
In the background we briefly mentioned that username/password logins are only one approach to authentication. While other authentication mechanisms are commercially available—for example, there are laptops with thumbprint scanners—for most web-based applications, username/password combinations are the most practical approach. It's simply unrealistic to assume that general web users will have card readers, key fobs, biometric scanners and so forth. Such assumptions may be more plausible in controlled environments.

It's useful to understand some of the drawbacks of username/password authentication. The main drawback is that in general, the stronger a user's password is, the harder it is to remember. That in turn increases the chance that the user will either write it down somewhere or else use it for multiple websites, both of which increase the chance that the password will be compromised. Password reuse is problematic partly because a shared password is distributed across many systems (which can be dangerous if that password is not properly managed; see recipes 4.3 and 4.5), and partly because the damage is not limited in the event that the shared password is compromised.

People have devised different approaches to dealing with parts of this problem. One approach is called single sign-on (SSO), and it works nicely in homogeneous computing environments such as corporate intranets. The idea is that the environment (rather than the application) provides for centralized authentication and then that authentication is shared with the applications in that environment. Another approach is the relatively recent OpenID standard, which differs from SSO in being decentralized. With OpenID, a user picks a participating site to be an authentication provider, and then other participating sites use that provider to authenticate the user. This is different than SSO in that with OpenID, different sites require separate logins. While neither SSO nor OpenID does much to limit the damage when a password is compromised (though a password could be reset from a single location), they do have the benefit that they prevent password fatigue without distributing the password across multiple systems.

Remember-me authentication also highlights the tradeoff between security and usability. It's important to recognize that we're really authenticating the browser rather than the user. Because there are various scenarios in which there's a many-to-one relationship between users and browsers (for example, a shared public machine, the family computer, a computer in somebody's unlocked office), we should treat remember-me authentication as not-entirely-convincing. One good approach is to give a remember-me user access to low-risk functionality, but to force a login if the user tries to access higher-risk functionality. Amazon uses exactly this technique: remember-me users can see product recommendations, but to buy something they have to log in.

The upshot is this: add login forms and remember-me to your app only if it makes sense to do so. In some cases it would make even more sense to use SSO, auto-provision the accounts, externalize your authentication with OpenID, and so on. Just because Spring Security makes it easy to add a login form and remember-me doesn't mean that you should. If you include remember-me in particular, keep in mind the design issues we just discussed.

Now that we've seen how to create a basic, unstyled login form, it's time to get to work on making it look better. The next recipe will show you how.</blockquote>