---
layout: post
title: "Hashing and salting passwords with Spring Security 2"
date: 2008-10-11 19:02:44
comments: true
categories: [Chapter 06 - Authentication, Tutorials]
---
<div class="intro"><span class="icon stickyNote">This post was originally written as a recipe for our book <a href="http://www.manning.com/wheeler/">Spring in Practice</a>, but we just didn't have enough room to include it. It's still (we think, anyway) a great recipe, so we're making it available here free of charge. This will give readers of this website a chance to see the sort of recipe we're including in the book.</span>

This material is based upon another post called <a href="http://springinpractice.com/2008/08/31/storing-passwords-securely/">Storing passwords securely</a>.</div>

<h3>Prerequisites</h3>
<a href="http://www.manning.com/wheeler/">4.3 Save user registrations</a>

<h3>Key Technologies</h3>
Spring Security, hash functions (e.g., MD5, SHA-1)

<h2>Background</h2>

When you're building an application involving user passwords, one of the challenges is to protect the passwords from prying eyes. We generally want to protect user registration data in transit from the browser to the server to prevent eavesdroppers on the network from getting at that data. But it's not enough to protect the data in transit. It's as important to store it securely.

It may not be immediately clear why you need to store the passwords securely. If, for instance, you have a corporate firewall that prevents the bad guys on the outside from getting to your password store, then why bother?

There are a few different answers to that. First, one basic security principle is to take a layered approach. There are no security silver bullets, and so overreliance on any particular process or technology (such as firewalls) creates unnecessary risk. Storing passwords securely gives you protection if a bad guy somehow gets in. (And it happens.)

A second answer is that the question makes the faulty assumption that the bad guys are on the outside. In an organization of any size, you can't be sure that you don't have bad guys on the inside, and the last thing you want them to do is use user passwords to log into your websites, or even other websites. The latter is a real concern since many users tend to use the same (or similar) passwords on multiple sites.

A third answer is that even if your organization is small (maybe it's just you), you'd like to be able to tell your users in your privacy policy that their passwords are stored securely, and that no one&mdash;not even the technical folks who administer the site&mdash;has any ability to see the stored passwords. That gives your users greater confidence that their information is safe and that you're taking security seriously.

So let's get to it!

<h2>Problem</h2>

You want to store passwords securely to prevent anybody&mdash;even yourself and other technical staff&mdash;from being able to view them.

<h2>Solution</h2>

In this recipe we're going to use Spring Security 2 to store passwords securely. We'll also show how to authenticate against secured passwords.

Spring Security makes it fairly easy to store passwords securely. The actual storage mechanism (database, LDAP or other) doesn't really matter since that's abstracted away, which is nice.

Before we jump into the how-to part of the recipe, let's briefly examine hash functions and how they work, since we'll need this background to understand what we're doing in the subsequent discussion.

<h3>Understanding hash functions</h3>

The idea behind this recipe is that we want to store passwords in encrypted form in the persistent store. There are in general two different approaches to encryption here: ciphers and hash functions. The two are similar in that each provides a way to encrypt data, but they differ in that only ciphers provide an easy way (assuming you know a secret key) to decrypt the data. Hash functions don't provide for easy decryption; hence they are sometimes referred to as one-way hash functions. All hash functions, however, are one-way.

When storing encrypted passwords, the typical practice is to use a hash function, such as MD5 or SHA-1, to encrypt the password before saving it. The reason for preferring a hash function to a cipher is that we don't usually want anybody to be able to recover the password: we never (or should never) display it on the screen or in e-mails, for example. A string of plaintext encrypted by a hash function is called a <em>hash</em>.
	
Here are a couple of examples of hashed text:

<ul>
<li><code>4fe0e930dd0adf9daaba0b7219bbe1f1bbb7809c</code></li>
<li><code>3af00c6cad11f7ab5db4467b66ce503e</code></li>
</ul>

The first string is an SHA-1 hash of the string <code>dardy</code>, and the second is an MD5 hash of the string <code>friend</code>. You can easily Google for free online hash calculators, both for MD5 and for SHA-1.
	
You might ask how we can use stored password hashes to authenticate users if we can't recover the plaintext password from the hash. The answer is that we don't compare plaintext passwords, but rather we compare the hashes. When the user submits a username/password pair, we hash the submitted password and compare that hash with the stored hash. If the two match, then we conclude that the submitted password was correct.

We can draw this conclusion due to a special property of hash functions: in general, different plaintext inputs map to different hashes. That is, in general, different plaintext inputs very, very rarely "collide." There are of course limits to this non-collision, but for practical purposes we can assume that different passwords will have different hashes. So if the hashes match, then the submitted password was correct.

Sound good? Let's start with the authentication side of things, and then we'll tackle user registrations.

<h3>Configuring your app for authentication against secured passwords</h3>

When discussing secure storage of passwords, there are really two different sides to consider. First, during the registration process (or else during account provisioning, if that's how you're doing things), you have to obfuscate the password in some fashion before saving it to the data store. And secondly you have to be able to use those obfuscated passwords during authentication.

We're going to start with the authentication part first, since that's pretty easy to set up. We're going to assume the following:

<ul>
<li>Your app has an existing user account model is called <code>Account</code>, with a corresponding <code>AccountDao</code> interface and implementation to support CRUD operations.</li>
<li>You also have a service bean, which we'll assume is <code>AccountServiceImpl</code>, that supports user registrations. The registrations store the passwords as plaintext.</li>
<li>Your app is configured to support Spring Security 2 username/password authentication against plaintext passwords. You're using two separate application context files: <code>applicationContext.xml</code> and <code>applicationContext-security.xml</code>, and you're using namespace configuration in <code>applicationContext-security.xml</code>.</li>
</ul>

Given the assumptions above, it isn't hard to modify the configuration to use hashed passwords during the login process. Let's see how.

<h3>Create some user accounts in the database with hashed passwords</h3>

Create an account or two in your database with SHA-1 hashed passwords. You can use the following website to compute SHA-1 hashes of plaintext passwords:

<a href="http://sha1-hash-online.waraxe.us/">http://sha1-hash-online.waraxe.us/</a>
	
For example, the SHA-1 hash of <code>flower</code> is <code>5a46b8253d07320a14cace9b4dcbf80f93dcef04</code>. When entering your hash into the hash calculator at the link above, be sure not to enter a carriage return character, as that's significant and will completely change the hash. Anyway, create the database records, using the SHA-1 hash as the password.
	
<h3>Update applicationContext-security.xml to authenticate against hashed passwords</h3>

We want to add something called a <code>PasswordEncoder</code> to our Spring configuration. <code>PasswordEncoder</code> is an interface defining a contract for computing hashes. There are different <code>PasswordEncoder</code> implementations according to the hash function you want to apply. Two popular options, for example, are <code>Md5PasswordEncoder</code> for MD5 hashes and <code>ShaPasswordEncoder</code> for (you guessed it) SHA. We'll use SHA here but either choice is legitimate. Don't use <code>Md4PasswordEncoder</code> unless you are working with a legacy system, as it is known to be weak.
	
The namespace configuration supports <code>PasswordEncoder</code>s. Just add a single line to your <code>authentication-provider</code> element like this:
	
<pre>&lt;authentication-provider&gt;
	&lt;password-encoder hash="sha" /&gt;
	&lt;jdbc-user-service data-source-ref="dataSource" /&gt;
&lt;/authentication-provider&gt;</pre>

Yep, that's it! This configuration sets SHA-1 as our hash algorithm.

<h3>Try it out</h3>

Try logging in using the account you just created. Type in the plaintext password, not the hash. The login should work. If so, congratulations, your app authenticates against secure passwords! (You'll need to convert whatever other passwords you have into SHA-1 hashes.)

Now we're going to jump over to the other side, which is storing passwords securely during either the registration or account provisioning process. We'll assume a registration process but the technique is the same in either case.

<h3>Configuring your app to secure passwords during registration</h3>

Saving secured passwords is somewhat more involved than authenticating against them. Part of the reason is that we'd like to use the same <code>PasswordEncoder</code> instance during registration that we used during login, partly just because it's "cleaner", and relatedly, because it allows us to avoid having to coordinate two separate <code>PasswordEncoder</code> instances should we decide to switch from SHA-1 to MD5, or add salt, or whatever. Basically it boils down to our being good adherents to the <a href="http://en.wikipedia.org/wiki/DRY_code">DRY principle</a>.

Unfortunately, the namespace configuration for <code>authentication-provider</code> doesn't accept an externally-defined <code>PasswordEncoder</code> bean; you have to use the <code>password-encoder</code> namespace element inside of <code>authentication-provider</code>. So instead of using the namespace configuration, we're going to drop back to good, old-fashioned bean configuration.

<pre>&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;beans:beans xmlns="http://www.springframework.org/schema/security"
    xmlns:beans="http://www.springframework.org/schema/beans"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
        http://www.springframework.org/schema/security
        http://www.springframework.org/schema/security/spring-security-2.0.4.xsd"&gt;

    &lt;-- 1 --&gt;
    &lt;beans:bean id="accountDao" class="example.HbnAccountDao" /&gt;
    
    &lt;-- 2 --&gt;
    &lt;beans:bean id="userDetailsService"
        class="example.UserDetailsServiceImpl"
        p:userDao="accountDao" /&gt;

    &lt;-- 3 --&gt;
    &lt;beans:bean id="passwordEncoder"
        class="org.springframework.security.providers.encoding.ShaPasswordEncoder" /&gt;

    &lt;-- 4 --&gt;
    &lt;beans:bean
        class="org.springframework.security.providers.dao.DaoAuthenticationProvider"
        p:userDetailsService-ref="userDetailsService"
        p:passwordEncoder-ref="passwordEncoder"&gt;
        &lt;custom-authentication-provider /&gt;
    &lt;/beans:bean&gt;
&lt;/beans:beans&gt;</pre>

In the code above, we begin by creating a data access object for user accounts <span class="cueball">1</span>. Next we create an implementation of Spring's <code>UserDetailsService</code> interface, which we've called <code>UserDetailsServiceImpl</code> <span class="cueball">2</span>. <code>UserDetailsService</code> is essentially a service provider interface for the <code>DaoAuthenticationProvider</code>; it allows the latter to obtain <code>UserDetails</code> instances for authentication purposes. Third we create the <code>PasswordEncoder</code> itself <span class="cueball">3</span>; in this case we're using <code>ShaPasswordEncoder</code>, which hashes passwords using SHA (the default strength is SHA-1). Finally, we're creating a <code>DaoAuthenticationProvider</code> <span class="cueball">4</span> and injecting it with the <code>userDetailsService</code> and <code>passwordEncoder</code> we created previously. We also include a <code>custom-authentication-provider</code> element to register our authentication provider with an <code>AuthenticationManager</code> hiding in the background.

<img src="http://wheelersoftware.s3.amazonaws.com/articles/spring-security-hash-salt-passwords/authn.jpg" alt="Authentication" />

For the curious, the <code>AuthenticationManager</code> in question is <code>ProviderManager</code>, a provider-based implementation of the <code>AuthenticationManager</code> interface, and it is automatically created by the <code>http</code> element unless you've already defined one explicitly. See the figure above for a class diagram.

That takes care of configuration. Let's see what we need to do in order to save registrations with a hashed password.

<h3>Update AccountServiceImpl to hash the password</h3>

There are two updates you'll need to make to your <code>AccountServiceImpl</code> bean. The first one is that you'll need to provide a setter for a <code>PasswordEncoder.</code> And once you create that setter, go back to your application context configuration and make sure you're actually injecting a <code>PasswordEncoder</code> into the <code>AccountServiceImpl</code> bean.
	
The second update is to modify your <code>registerAccount()</code> method in <code>AccountServiceImpl</code> so that it hashes passwords:
	
<pre>public void registerAccount(Account account) {
    accountDao.save(account); // 1
    String encPassword =
        passwordEncoder.encodePassword(account.getPassword(), null); // 2
    account.setPassword(encPassword); // 3
    accountDao.save(account); // 4
}</pre>
	
First we save the account with the plaintext password <span class="cueball">1</span>. There's a reason for saving the account before actually hashing the password, but we'll have to wait until later in the recipe for the reason why. Then we hash the password <span class="cueball">2</span>, update it on the account <span class="cueball">3</span>, and save the account with the new password <span class="cueball">4</span>. The account now has a hashed password in the database.
	
Good job! You're now saving secured passwords into your database, and you're able to authenticate against them.

But I'm afraid that it's time for some bad news.

<h3>Meet the dictionary attack</h3>

Our new password storage scheme is off to a good start. It certainly prevents casual observers from accidentally seeing user passwords. But it does little to thwart the efforts of a semi-determined attacker. Let's talk about the dictionary attack.

To understand how it works, recall that hashing different strings generally results in different hashes. We can use that fact to create a big lookup table for a dictionary of potential passwords. The lookup table takes a hash and then returns the string that generated it.

As luck would have it, we don't even have to create our own lookup table. Helpful folks on the Internet have already done it for us. For instance, go to

<a href="http://tools.benramsey.com/md5/">http://tools.benramsey.com/md5/</a>

and enter the MD5 hash we presented earlier; namely,

<code>3af00c6cad11f7ab5db4467b66ce503e</code>
	
Voil&agrave;! You've unhashed a hash. This is called a <em>dictionary attack</em>.

If an attacker were to acquire a list of hashed passwords, he could make quick work of it using a dictionary of the sort just described. In some contexts that might be very bad.

<h3>Add a little salt to that hash</h3>

Ideally we'd like to thwart even the semi-determined attacker. To improve our scheme, we now introduce the idea of salt.

The strategy behind salt is to make it much more painful for the attacker to recover hashed passwords. Instead of allowing ourselves to be attacked by a single well-known dictionary, we are going to force the attacker to create a new and unique dictionary for every single password he wants to try to recover. Can he still do it? Sure. But it's just a lot more work now. We've effectively eliminated merely semi-determined attackers from the pool of attackers. That's nothing to scoff at since their numbers are large.

Here's how we do it. Instead of hashing passwords, we concatenate a string&mdash;called a salt&mdash;to the plaintext password, and then hash the concatenated string. This effectively breaks the standard dictionary attack, because now all of the hashes in your password store are "new."

What happens, though, if we use a single, common salt across all users? While this is better than using no salt at all, it's still not hard to overcome if the attacker knows the salt. The attacker simply has to create a single new dictionary, this time concatenating the salt to each individual dictionary word before hashing. Then it's the same as before.

We want to make things harder for him. Instead of using a common salt across all users, we want the salt to be different for each user. That way, the attacker has to create a new dictionary for every single user he wants to attack. Again, he can do it, but it's much more time-consuming.

One good way to create a salt is to use some property of the user. It is better to choose something that won't change, such as a numeric primary key, than it is to choose something that might change, such as an e-mail address. If you use (say) e-mail addresses for salt, and a user changes his e-mail address, he won't be able to log in anymore because the authentication system won't be able to create the correct hash.
	
Usernames are a reasonable choice as they don't usually change, but a numeric primary key is even better, given the specific scheme Spring Security uses to concatenate the password with the salt. Spring Security uses braces to delimit the salt, and hence disallows braces inside the salt itself. For example, if my password/salt is <code>college/willie</code>, then Spring Security will hash the string <code>college{willie}</code>. You may well want to allow braces in the username (it's common for gamers to include such characters in their usernames). You can avoid the whole issue&mdash;including the added complexity of disallowing braces in usernames&mdash;by using the user's numeric primary key, if the user schema supports that.
	
So let's do just that.

<h3>Update applicationContext-security.xml to handle salt</h3>

All we need to do to <code>applicationContext-security.xml</code> is add a <code>SaltSource</code> bean and inject it into the <code>DaoAuthenticationProvider</code>. We do this in the listing below.
	
<pre>&lt;beans:bean id="passwordEncoder"
    class="org.springframework.security.providers.encoding.ShaPasswordEncoder" /&gt;
&lt;beans:bean id="saltSource"
    class="org.springframework.security.providers.dao.salt.ReflectionSaltSource"
    p:userPropertyToUse="id" /&gt;
&lt;beans:bean
    class="org.springframework.security.providers.dao.DaoAuthenticationProvider"
    p:userDetailsService-ref="userDetailsService"
    p:passwordEncoder-ref="passwordEncoder"
    p:saltSource-ref="saltSource"&gt;
    &lt;custom-authentication-provider /&gt;
&lt;/beans:bean&gt;</pre>
	
We mentioned above that it's possible to use a global salt, but that it's not as good as using a salt that varies from user to user. (For a more detailed explanation, please see my article <a href="http://springinpractice.com/2008/08/31/storing-passwords-securely/">Storing Passwords Securely</a>.) <code>ReflectionSaltSource</code> allows us to use a property of our user (specifically, a property of a <code>UserDetails</code> instance; we'll see that shortly) to provide a salt. As with the <code>PasswordEncoder</code>, we've defined the <code>SaltSource</code> explicitly as a bean so we can inject it into <code>AccountServiceImpl</code>.
	
<h3>Update AccountServiceImpl to salt the password</h3>

The first step is to add a setter for a <code>SaltSource</code> instance to <code>AccountServiceImpl</code>, and wire it up in your application context file.
	
Next, we'll once again update <code>registerAccount()</code>.
	
<pre>public void registerAccount(Account account) {
	accountDao.save(account);
	UserDetailsAdapter userDetails = new UserDetailsAdapter(account); // 1
	String password = userDetails.getPassword();
	Object salt = saltSource.getSalt(userDetails); // 2
	account.setPassword(passwordEncoder.encodePassword(password, salt)); // 3
	accountDao.save(account);
}</pre>

As before, we're saving the account with a plaintext password first. I promised earlier to explain why I'm doing that, so here's the explanation. In many cases (such as when using Hibernate), entities aren't assigned IDs until after they're persisted. Since we're basing our salt on the ID, we need to save the account before generating the salt and hash.

Next we wrap the <code>account</code> with a class we wrote to adapt our <code>Account</code> class to the <code>UserDetails</code> interface; namely, <code>UserDetailsAdapter</code> <span class="cueball">1</span> (we'll see it momentarily).
	
We're using the <code>SaltSource</code> to get the salt from our <code>UserDetailsAdapter</code> <span class="cueball">2</span>. As we saw in the configuration, this salt source uses reflection to grab the account ID and present it to the <code>PasswordEncoder</code> as salt <span class="cueball">3</span>.
	
The next listing shows our <code>UserDetailsAdapter</code> class, which again is just an example of a <code>UserDetails</code> implementation. I'm extending the <code>org.springframework.security.userdetails.User</code> class for convenience, but note that that's a <code>UserDetails</code> implementation.
	
<pre>package examples;

import java.util.Set;

import org.springframework.security.GrantedAuthority;
import org.springframework.security.GrantedAuthorityImpl;
import org.springframework.security.User;

public class UserDetailsAdapter extends User { // 1
	private final Long id;
	
	public UserDetailsAdapter(Account acct) {
		super(acct.getUsername(), acct.getPassword(), acct.isEnabled(),
				true, true, true, toAuthorities(acct.getAuthorityNames()));
		this.id = acct.getId();
	}
	
	private static GrantedAuthority[] toAuthorities(Set&lt;String&gt; authNames) {
		GrantedAuthority[] auths = new GrantedAuthority[authNames.size()];
		int i = 0;
		for (String authName : authNames) {
			auths[i++] = new GrantedAuthorityImpl(authName);
		}
		return auths;
	}
	
	public Long getId() {
		return id;
	}
}</pre>
	
Again, we're just extending <code>User</code> for convenience <span class="cueball">1</span>; the important thing is that our <code>UserDetailsAdapter</code> implements <code>UserDetails</code>, which allows <code>DaoAuthenticationProvider</code> to use it to perform authentication.
	
There you have it&mdash;salted and hashed passwords. This won't stop the most determined attackers from unmasking your passwords, but it will stop most others.

<h2>Discussion</h2>

The ideas in this recipe can also be applied to storing answers to challenge questions in a secure way (for instance, "What is your mother's maiden name?"). The relevant similarities are that the information should be secured (otherwise it might be used to recover or reset passwords on other websites), and that we typically don't need to decrypt answers to challenge questions. It might be helpful to convert the answers to some kind of canonical form before saving them, such as converting everything to uppercase, removing punctuation and trimming extra whitespace, just to eliminate minor variations in the way that people answer the question.

It is also useful to understand where this recipe does not apply. In most cases where you want to store data securely, you need to be able to decrypt the stored data. A good example is storing credit card numbers securely for Payment Card Industry (PCI) compliance. You need to be able to recover the credit card number so you can apply it to customer orders. One-way hashes, which support encryption but not decryption, will not help you with this.

When you store passwords using hashes, you cannot offer password recovery functionality to end users, because the system doesn't have any way to actually recover the passwords. All you can offer is the ability to reset passwords. This is better from a security perspective anyway since there's always the chance that you might disclose a password to somebody other than the intended recipient, and that somebody might use it to access not only your site but other sites, as we discussed in the background above.

<h2>Resources</h2>

If you found this article helpful, you may also find the following useful:

<ul>
<li><a href="http://springinpractice.com/2008/08/31/storing-passwords-securely/">Storing passwords securely</a> - Another post I wrote about storing passwords securely, but treating the subject more generally as opposed to being Spring-specific. The discussion about hashing and salting is a little more in-depth, and there's also a section on key strengthening, which can make your passwords even more secure.</li>
<li><a href="http://www.manning.com/wheeler/">Spring in Practice</a> - A book that my collaborators and I are writing for Manning Publications about Spring generally. We have lots of information about Spring Security, including chapters on user registrations, authentication and authorization.</li>
<li><a href="http://springinpractice.wordpress.com/2008/09/06/login-remember-me/">Excerpt: Login and remember-me discussion</a> - An excerpt from Spring in Practice that compares normal username/password authentication with remember-me authentication.</li>
</ul>

<div class="endnote">Post migrated from my Wheeler Software site.</div>

