---
layout: post
title: "Storing passwords securely"
date: 2008-08-31 10:28:35
comments: true
categories: [Chapter 06 - Authentication, Tutorials]
---
When dealing with user account information, there are lots of different security concerns that come up. Some examples include making sure users use strong passwords, preventing automated registrations, helping end users distinguish real sites from phishing sites, transmitting user data securely, and so forth&mdash;the list goes on and on.

In this article we're going to look at a specific piece of the overall user account security puzzle, and that's the problem of storing user passwords securely. Specifically we will see how you can use techniques from cryptography&mdash;hashing and salting your passwords&mdash;to make it harder for would-be attackers to uncover user passwords in the event that the password database (or password file, or whatever) is compromised.

We're only going to cover the concepts and techniques here; we won't get into how to actually implement this on any particular platform or using any particular framework. My brother John and I are currently writing a book for Manning Publications called <a href="http://www.manning.com/wheeler/">Spring in Practice</a> that will explain how to implement password salting and hashing using the <a href="http://www.springframework.org/">Spring Framework</a>, but for this article we'll just look at the concepts and techniques.

<h3>Why store user passwords securely?</h3>

It's worth thinking for a moment whether it's really even necessary to "store user passwords securely," if doing so involves anything more than limiting access to password data. If only a small number of trustworthy technical staffers have access to the user passwords, then why can't we just store the data in the clear? Doesn't that make it easier for technical staff to troubleshoot system issues (for example, admins can just log in as other users to try to reproduce errors), and doesn't that make it easier to help users recover forgotten passwords?

The answer to both of those questions is yes, but the story doesn't end there. The problem is that storing plaintext passwords also makes it easier for bad guys to see user passwords, and that's true whether bad guys break in from the outside or whether you have bad guys on your admin team. If attackers can log into your applications under somebody else's account, that's potentially very bad for you and your users. Moreover, because users often have the bad habit of reusing passwords (or else simple variants of passwords) across multiple applications and web sites, it's very difficult for you to limit the scope of the damage once somebody actually gets ahold of user passwords. The situation is possibly very damaging to your users and embarrassing to your organization. In all but the most trivial cases, it makes a lot of sense to store user data securely.

As an aside, note that in doing so, we don't give up the ability to allow admins or tech support reps to impersonate other users, nor do we sacrifice the ability to help users who have forgotten their passwords. We just have to use other techniques to accomplish those things, such as "run-as" impersonation, and using password resets in lieu of password recovery. Let's however stay focused on storing passwords securely.

So now we've seen why we don't want to store plaintext passwords. In the rest of this article we're going to look at four increasingly sophisticated cryptographic techniques for making it harder for attackers to get at your passwords. All four are based on something called a hash function, so let's start by talking about hash functions a bit.

<h3>Understanding hash functions</h3>

Hash functions are a tool from cryptography, and they are functions in the mathematical sense: for any given "acceptable" input, it spits out a specific output. In this case, the domain of acceptable inputs would be plaintext strings, and outputs are garbled-up (i.e., encrypted) versions of the plaintext strings, called <em>hashes</em>.

Let's play around with some examples before we go on, just so you can see what I'm talking about. Go to your favorite search engine and type "online hash function calculator." (Obviously you can use command line tools and programming APIs to compute hash functions as well.) You should see some links that allow you to calculate hashes using specific hash functions with names like MD5, SHA-1, RIPEMD-160 and others. Here's one:

<ul>
 <li><a href="http://www.hashemall.com/">http://www.hashemall.com/</a></li>
</ul>

Select an MD5 hash calculator, and enter some plaintext. When you hit submit, you should see a hash value. Note that whitespace in your plaintext is significant, so if you hash 'friend' and 'friend[CR]', the output will be different.

Here are some MD5 hashes:

<table>
 <tr>
  <th style="width:40%">Plaintext input</th>
  <th style="width:60%">MD5 hash (hex)</th>
 </tr>
 <tr>
  <td><code>friend</code></td>
  <td><code>3af00c6cad11f7ab5db4467b66ce503e</code></td>
 </tr>
 <tr>
  <td><code>friends</code></td>
  <td><code>28f20a02bf8a021fab4fcec48afb584e</code></td>
 </tr>
 <tr>
  <td><code>password</code></td>
  <td><code>5f4dcc3b5aa765d61d8327deb882cf99</code></td>
 </tr>
 <tr>
  <td><code>I pledge allegiance to the flag</code></td>
  <td><code>bb00ea10b4ee04c4319c0c05bf9c29fc</code></td>
 </tr>
</table>

Just for kicks let's get some SHA-1 hashes for the same plaintext inputs:

<table>
 <tr>
  <th style="width:40%">Plaintext input</th>
  <th style="width:60%">SHA-1 hash (hex)</th>
 </tr>
 <tr>
  <td><code>friend</code></td>
  <td><code>e69867ca7d5a7b0ab60a2a61e7b791c106f7bf64</code></td>
 </tr>
 <tr>
  <td><code>friends</code></td>
  <td><code>3d9209c4598bfbc38b3c096081bee3a09697e939</code></td>
 </tr>
 <tr>
  <td><code>password</code></td>
  <td><code>5baa61e4c9b93f3f0682250b6cf8331b7ee68fd8</code></td>
 </tr>
 <tr>
  <td><code>I pledge allegiance to the flag</code></td>
  <td><code>fc1d13f4e3b942d6c31185ff033c5b7acfe22751</code></td>
 </tr>
</table>

There's a lot of good stuff going on here. The main point is that MD5 and SHA-1 both do essentially the same thing: they convert plaintext into garbled text with certain properties (which we'll discuss momentarily). So they're both hash functions, albeit different hash functions.

I just mentioned "certain properties." Here are some worth noting:

<ul>
 <li>All of the MD5 hashes are the same length as each other, and similarly, all of the SHA-1 hashes are the same length. MD5 hashes use 128 bits (usually expressed as 32 hex digits) and SHA-1 hashes use 160 bits (usually expressed as 40 hex digits). In general hash functions (or more exactly, cryptographic hash functions) return fixed-size hashes.</li>
 <li>It was really easy to compute the hashes from the inputs. If you used one of the online calculators, for example, your hash value was calculated more or less instantly after you provided the plaintext.</li>
 <li>Looking at the hashes, there's no obvious way to map it back to the plaintext. Ideally it is computationally infeasible to determine the plaintext input from the hash.</li>
 <li>Similar inputs, such as 'friend' and 'friends', produce dramatically different hashes.</li>
</ul>

The properties just given are characteristic of hash functions. For a hash function to be "cryptographically secure," we typically want two more properties to be true:

<ul>
 <li>Given a hash, it is computationally infeasible to find an input that produces that hash.</li>
 <li>Given an input, it is computationally infeasible to find another input that produces the same hash.</li>
</ul>

The MD5 and SHA-1 hash functions are generally considered to be secure though researchers have found weaknesses in each. For many practical purposes, it is at the time of this writing (late 2008) safe to use either of the two. <strong>If however you are doing something that requires exceptionally strong security (for example, financial applications), please consult a security expert.</strong>

That's what we needed to know about hash functions. Now let's learn how we can use them to store passwords securely.

<h3>Using hashes to protect passwords from casual observers and undetermined attackers</h3>

The first thing we might reasonably want to do is protect passwords from casual observers, such as technical staff who have access to passwords in the database. While your admins are probably people you trust, there's no reason to tempt them, right?

The easiest way to protect passwords from casual observers is to hash the passwords before storing them in the database. That way admins see only hex codes instead of plaintext passwords.

If you recall from the previous section, we mentioned that in general we can't compute the password from the hash. You might wonder then how we accomplish logins. The answer to that is to hash the user's submitted password when he logs in, and then to compare the submitted hash with the stored hash. Because different passwords will almost certainly have different hashes, we can conclude that the authentication was successful if and only if the two hashes match. Sounds great, right?

Not exactly. This scheme does in fact prevent the casual observer from accidentally seeing passwords, and it may even present the newbie attacker from recovering a password. But in jujutsu-like fashion, we can use the for-all-practical-purposes 1-1 nature of the hash function as a tool to defeat it. Let's look at an example, shall we?

<ol>
 <li>Go to Google and search for "online md5 hash database". Here are a couple that I found. (Note that I don't have any control over these websites and in particular I can't promise that they won't have unsavory ads. These two don't appear to have that, but sometimes hacker websites do have that sort of thing.)
  <ul>
   <li><a href="http://md5.igrkio.info/md5-hash-database.html">http://md5.igrkio.info/md5-hash-database.html</a> (this is a metasearch)</li>
   <li><a href="http://gdataonline.com/seekhash.php">http://gdataonline.com/seekhash.php</a></li>
  </ul>
 </li>
 <li>Copy the MD5 hash for 'friend', 'friends' or 'password' above.</li>
 <li>Paste it into the hash database search field submit it.</li>
 <li>Hey, you've recovered your password! Huh?!</li>
</ol>

So what's going on here? It turns out that there are various online hash databases&mdash;essentially lookup tables&mdash;that map hashes to the inputs that generated them. Helpful hackers around the world create such databases by precomputing the hashes associated with dictionary words and close variants (in multiple languages, even). Because different passwords map to different hashes, you can create a lookup table for any explicitly given set of passwords. If an attacker has such a hash database at his disposal, it's pretty easy to launch a so-called <em>dictionary attack</em> against a compromised password database. The attacker can either attack passwords one-by-one, or (more commonly), the attacker can simply look for hashes associated with common passwords ('password', 'qwerty', etc.) and do whatever he wants to do with them afterward.

You'll notice that we did not pick the hash for 'I pledge allegiance to the flag'. If you copy the MD5 hash for 'I pledge allegiance to the flag' into a hash databases, chances are that it won't be able to figure out what generated the hash. (And if it does, it's easy enough to manufacture hashes that won't generate a match.) That's because the hash database creator did not precompute a hash for that particular phrase. And we can use that little tidbit to make our hashing scheme more secure, as we're about to see.

<h3>Using hashes and fixed salts to protect passwords from slightly-determined attackers</h3>

The key insight from the above is that we can thwart simple dictionary attacks by hashing inputs that are unlikely to have been used to generate a hash dictionary. One possibility would be to ask users to use strong passwords with a mix of uppercase, lowercase, digits and punctuation, but we don't want to have to depend on users to do that. (Anyway, the more complicated the passwords are, the more likely the user is to write the password down on a Post-It, which creates its own security issues.) So what else can we do?

The answer is that we can concatenate the plaintext with some given additional piece of text, known as a "salt," and then hash and store the result. The salt should be something that you're pretty sure wouldn't be used in generating a generic hash database. That way there's no chance that somebody could use an existing dictionary to attack your passwords.

Let's say, for instance, that we choose <code>iamJeLLy%Dans0R</code> as our salt. Then we can generate safer hashes as follows:

 <table>
  <tr>
   <th>Plaintext password</th>
   <th>Password + salt</th>
   <th>MD5 hash of password + salt</th>
  </tr>
  <tr>
   <td><code>friend</code></td>
   <td><code>friend{iamJeLLy%Dans0R}</code></td>
   <td><code>a62dc48775f623e180dd0ff2bc870b24</code></td>
  </tr>
  <tr>
   <td><code>friends</code></td>
   <td><code>friends{iamJeLLy%Dans0R}</code></td>
   <td><code>ec844e0a820ae2e4444f70c5a447c467</code></td>
  </tr>
  <tr>
   <td><code>password</code></td>
   <td><code>password{iamJeLLy%Dans0R}</code></td>
   <td><code>1d7eed225b27a9cd075478e7d5de29a6</code></td>
  </tr>
  <tr>
   <td><code>I pledge allegiance to the flag</code></td>
   <td><code>I pledge allegiance to the flag{iamJeLLy%Dans0R}</code></td>
   <td><code>cd7e0f644ebecb7c7c3558c71a8f7a86</code></td>
  </tr>
 </table>

Call me an optimist, but I am pretty sure you are not going to find any of those hashes in an existing hash database. So now we are bulletproof, yes?

No, not really. (And there probably isn't such a thing when it comes to security.) You may have noticed that I kept saying that the hashes wouldn't appear in an <em>existing</em> hash database. That's the clue you need in order to figure out how the attacker can still get at your passwords. Assume also that the attacker knows the salt value, which isn't necessarily unrealistic since he does after all have access to your passwords. Maybe he has access to the salt too. Do you see the attack?

All the attacker needs to do is generate a new lookup table. He grabs a file containing hundreds of thousands of dictionary words, appends each one with the salt, computes the hash and finally creates the lookup entry. Ugh!

How realistic is that? For casual observers and novice attackers, it's unlikely. But if somebody is fairly determined to get the passwords, it's not unlikely at all. Dictionary files in multiple languages are after all readily available&mdash;see, e.g., <a href="http://www.winedt.org/Dict/">http://www.winedt.org/Dict/</a>&mdash;and anyway, it's not even necessary to process the entire dictionary. Just choose say 1,000 common words and do those. That'll get you pretty far if there are enough passwords in the password database.

You might wonder why we would even look at this approach, given that it's easy enough to defeat. There are multiple reasons. First, even though it's easy to defeat, it does in fact create an extra obstacle to getting at the passwords, and that obstacle is sufficiently frustrating that it will filter out attackers who aren't at least fairly determined and knowledgeable. Secondly, we assumed that the salt was known. If you're able to keep the salt a secret (e.g., store it separately from the password database), then you have the makings of a practically impenetrable password storage scheme. The problem however is keeping the salt secret; presumably either the developers or the admins will know it. But if you're able to pull that off, go for it.

The third reason is that we can use a modified version of our salt approach to make password recovery even more difficult still. The trick is to use different salts for different passwords. Let's see how that works.

<h3>Using hashes and variable salts to protect passwords from fairly determined attackers</h3>

To make it even harder to recover passwords from hashes, we're going to force the attacker to create a brand new dictionary for every single password he wants to attack. We do this by using a variable salt instead of a fixed salt. We simply choose some property of the user whose password we want to protect and use that as the salt. For example, we might use the user's numeric primary key, or perhaps the username.

You can see that with this scheme there is no single dictionary that will support the attack. The attacker has to create a new dictionary for each password. The attacker has to have much more patience and be much more determined in order for this attack to work, because now the work is linear in the number of users instead of constant. Of course, the attacker can make his job easier by attacking only a subset of the users or else by focusing on some subset of the dictionary words, but in general the computing problem is more intensive (even though no more difficult logically speaking).

When choosing a property to use as the salt, take care not to choose something that might change. If, for example, you were to choose the user's e-mail address and the user subsequently changed the e-mail address, then the user would suddenly lose the ability to log in.

Our defenses are getting beefier and beefier, but still it's not entirely unlikely that somebody with enough patience and determination could crack our passwords. Can we make it even harder?

<h3>Using key strengthening to protect passwords from strongly determined attackers</h3>

There's a trick we can use called <em>key strengthening</em> or <em>key stretching</em>. The idea is to increase the time associated with recovering any given password so as to increase the amount of patience and determination required on the part of the attacker. It's simple to implement. Take the initial password + variable-salt combination (you could use a fixed salt too, but as we saw above, variable salts turn a constant-time problem into a linear-time problem), hash it iteratively some large number of times&mdash;say 1,000 times&mdash;and store the final result in the password database. The exact number of iterations is strongly dependent on the environment; you want the number of iterations to be small enough that it doesn't impact legitimate end users in a meaningful way when they try to log in (they shouldn't have to wait more than a second or two), but large enough that if you try to hash an entire dictionary, you're going to be waiting an awful long time. If you have lots of end users and logins, you also have to be careful that you don't introduce too much CPU load on your system since hashing is compute-intensive. (It's easy enough to imagine building out a compute farm if you really intend to get this serious about storing passwords securely.)

Can this scheme be broken? Yes, of course. But once again we've narrowed the circle of folks who are going to attempt it. Most attackers are going to look for a much easier target. Bad for the easier targets but good for you.

<h3>Summary</h3>

In this article we've considered a series of increasingly sophisticated techniques for protecting your user passwords from attackers. As often happens with engineering, it's important to recognize that there's a tradeoff you're confronting here&mdash;you have to decide how to balance simplicity against security. If you're trying to protect passwords for an academic department's website, it's probably overkill to use key stretching. If on the other hand you're trying to protect financial data, key stretching may be only the starting point for you.

It's important to recognize that security isn't a black-or-white affair. As it does in this case, it usually involves understanding where your likely threats are and then taking measures to protect against those, and perhaps even against threats that are even a little less likely but still plausible.

<div class="endnote">Post migrated from my Wheeler Software site.</div>
