---
layout: post
title: "How to reCAPTCHA your Java application"
date: 2008-03-13 14:42:36
comments: true
categories: [Chapter 04 - Web forms, Chapter 08 - Communicating, Tutorials]
---
reCAPTCHA is a novel CAPTCHA system developed by the School of Computer Science at my alma mater, Carnegie Mellon University. I won't explain its coolness here since they do a good job of <a href="http://recaptcha.net/learnmore.html" target="_blank">explaining that coolness themselves</a>. What I will do here, though, is explain how to get your Java app reCAPTCHAed very quickly. Note however that reCAPTCHA is not tied specifically to Java.

In this tutorial I'm using Spring 2.5 MVC with annotations, and Commons Validator, but you'll be able to follow this whether or not you're using Spring and Validator.

These instructions are based on the <a href="http://recaptcha.net/apidocs/captcha/client.html" target="_blank">instructions from the reCAPTCHA site</a>, but I'm focusing specifically on Java integration whereas the site makes you dig around a bit to get the information. Not too bad, but enough that there's value in my writing a Java-specific tutorial. :-)

<h3>Step 1. Get your account and key pair</h3>

First, go to <a href="http://recaptcha.net/" target="_blank">the reCAPTCHA web site</a> and create an account. As part of that account creation process you'll have to specify the domain your reCAPTCHA will be protecting. The reCAPTCHA site will will give you a key pair for that domain. The key pair allows you to authenticate your reCAPTCHA requests to the reCAPTCHA servers, as we'll see.

<h3>Step 2. Put the reCAPTCHA JavaScript in your app's form</h3>

<p>Here's the JavaScript you need to put in your form, meaning in between the &lt;form&gt; and &lt;/form&gt; tags. Put it wherever you would have normally put a CAPTCHA text box. This JavaScript will generate the reCAPTCHA box when users request the page:

<pre>&lt;script type="text/javascript"
    src="http://api.recaptcha.net/challenge?k=&lt;your_public_key&gt;"&gt;
&lt;/script&gt;
&lt;noscript&gt;
    &lt;iframe src="http://api.recaptcha.net/noscript?k=&lt;your_public_key&gt;"
        height="300" width="500" frameborder="0"&gt;&lt;/iframe&gt;&lt;br&gt;
    &lt;textarea name="recaptcha_challenge_field" rows="3" cols="40"&gt;
    &lt;/textarea&gt;
    &lt;input type="hidden" name="recaptcha_response_field" 
        value="manual_challenge"&gt;
&lt;/noscript&gt;</pre>

It probably goes without saying, but I'll say it anyway: you need to replace the two instances of &lt;your_public_key&gt; with the public key that you received during the account creation process. Be careful that you don't use your private key by mistake. If you do that then everybody will be able to see your private key and act like they're you.

<h3>Step 3. Run your app and make sure the reCAPTCHA is showing up</h3>

You should see it there in your form. It's OK if you are coming from <code>localhost</code> or <code>127.0.0.1</code> instead of the domain that you specified in the account creation step; reCAPTCHA will allow that. You should be able to click the buttons on the reCAPTCHA box and they should work.

After you goof around with that a bit, you'll need to update your app itself so that it actually uses the reCAPTCHA box to validate the form submission.

Let's turn now to the Java piece, where we validate the form and reCAPTCHA.

<h3>Step 4: Validate the form, including the reCAPTCHA</h3>

You'll find it convenient to <a href="http://code.google.com/p/recaptcha/downloads/list?q=label:java-Latest" target="_blank">download the recaptcha4j library</a>. It provides a simple API for submitting user responses to the reCAPTCHA server and finding out whether a user's response is valid.

At this point I'm just going to lay some code on you. As mentioned above I'm using <a href="spring-mvc-annotations.html">Spring 2.5 MVC with annotations</a> and Commons Validator, but the main thing is for you to look at how I'm using the <code>ReCaptchaImpl</code> class and just copy that.

<pre>import net.tanesha.recaptcha.ReCaptchaImpl;
import net.tanesha.recaptcha.ReCaptchaResponse;
import org.springframework.validation.BindingResult;
import org.springframework.validation.FieldError;
import org.springframework.validation.Validator;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;

...

@RequestMapping(value = "/comments/postcomment.do", method = RequestMethod.POST)
public String doPost(
        HttpServletRequest req,
        @RequestParam("articleId") long articleId,
        @RequestParam("recaptcha_challenge_field") String challenge,
        @RequestParam("recaptcha_response_field") String response,
        @ModelAttribute("comment") Comment comment,
        BindingResult result) {
    
    // Validate the form (other than the reCAPTCHA)
    validator.validate(comment, result);
    
    // Validate the reCAPTCHA
    String remoteAddr = req.getRemoteAddr();
    ReCaptchaImpl reCaptcha = new ReCaptchaImpl();
    
    // Probably don't want to hardcode your private key here but
    // just to get it working is OK...
    reCaptcha.setPrivateKey("&lt;your_private_key&gt;");
    
    ReCaptchaResponse reCaptchaResponse =
        reCaptcha.checkAnswer(remoteAddr, challenge, response);
    
    if (!reCaptchaResponse.isValid()) {
        FieldError fieldError = new FieldError(
            "comment",
            "captcha",
            response,
            false,
            new String[] { "errors.badCaptcha" },
            null,
            "Please try again.");
        result.addError(fieldError);
    }
    
    // If there are errors, then validation fails.
    if (result.hasErrors()) {
        String path = comment.getPagePath();
        log.debug("Form validation error; forwarding to " + path);
        return "forward:" + path;
    }
    
    // Else validation succeeds.
    log.debug("Form validation passed");
    comment.setIpAddress(remoteAddr);
    comment.setDate(new Date());
    
    // Post the comment
    log.debug("Posting the comment");
    articleService.postComment(articleId, comment);
    log.debug("Comment posted");
    
    return "redirect:" + comment.getPagePath() + "#comments";
}</pre>

Here's the <a href="http://static.springframework.org/spring/docs/2.5.x/api/org/springframework/validation/FieldError.html" target="_blank">API for FieldError</a> since I know that's not clear from the code. Basically I'm using that to indicate that a validation error occurred and set up an error message for the user. If you're not using Spring/Validator then you'll do something else here.

The <code>Comment</code> class is just a class from my app, so don't worry about that one.

<h3>You did it</h3>

Good job. If you're feeling ambitious, try to defeat reCAPTCHA with super-advanced OCR. If you succeed then it represents an advance in OCR technology. Tell somebody and become famous. :-)

<div class="endnote">Post migrated from my Wheeler Software site.</div>