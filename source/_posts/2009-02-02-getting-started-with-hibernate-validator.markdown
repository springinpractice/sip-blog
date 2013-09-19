---
layout: post
title: "Getting started with Hibernate Validator"
date: 2009-02-02 15:00:16
comments: true
categories: [Chapter 04 - Web forms, Tutorials]
---
In this tutorial we're going to learn how to get started with Hibernate Validator, which as its name suggests is a validation framework associated with the Hibernate project. This article is really a follow-up to <a href="http://springinpractice.com/2008/07/17/annotation-based-validation-with-the-spring-bean-validation-framework/">my earlier article on using the Bean Validation Framework</a> (part of the larger <a href="https://springmodules.dev.java.net/">Spring Modules</a> project), which is a competing framework that seems to have lost the battle for Spring's "preferred validation provider" status to Hibernate Validator. Both Uri Boness (in an e-mail correspondence) and Juergen Hoeller (at SpringOne) agreed that people should start moving toward Hibernate Validator since that will eventually support the emerging <a href="http://jcp.org/en/jsr/detail?id=303">JSR 303</a> standard.

Hibernate Validator is nice because it (like the Bean Validation Framework) supports declarative validation via <a href="http://java.sun.com/j2se/1.5.0/docs/guide/language/annotations.html">Java 5 annotations</a>. Let's say you create a bean class, like an <code>Account</code> or a <code>PurchaseOrder</code> or whatever. With Hibernate Validator you can attach validation annotations to the bean properties and that will define the validation constraints for the bean. Moreover, unlike earlier approaches to validation (such as Struts Validation), Hibernate Validator isn't tied to the web tier, and so if you want to validate your beans from within your service beans, or within your DAOs, or even just before you ORM them into your database, no sweat. You can do just that.

Anyway, for now we're just going to look at some of the basics: how to specify annotation constraints and how to check for constraint violations. We're not going to worry about integrating Hibernate Validator with Spring's native validation framework (so that, for instance, we might render Hibernate Validator error messages out using Spring Web MVC taglibs) though I'll probably write another article on that sometime in the future if people are interested.</p>

For this article we're using Java 5 or higher (we need Java 5 annotations) and Hibernate Validator 3.1.0. For your convenience I've created a <a href="http://maven.apache.org/">Maven 2</a> project that you can download: <a href="http://wheelersoftware.s3.amazonaws.com/articles/hibernate-validator/hibernate-validator-demo.zip">hibernate-validator-demo.zip</a>

OK, let's jump into some examples of annotated bean classes.

<h3>Specifying validation constraints with Hibernate Validator annotations</h3>

Here are a couple of examples of annotated bean classes: a <code>User</code> class and an <code>Address</code> class. Note that the two objects are related in a parent-child fashion.

First, here is the <code>User</code> bean class:

<pre>package com.wheelersoftware.demos.hibernatevalidator;

import org.hibernate.validator.Email;
import org.hibernate.validator.Length;
import org.hibernate.validator.NotEmpty;
import org.hibernate.validator.Valid;

public class User {
    private String username;
    private String firstName;
    private String lastName;
    private Address address;
    private String email;
    private String password;

    @NotEmpty
    @Length(max = 20)
    public String getUsername() { return username; }

    public void setUsername(String username) {
        this.username = username;
    }

    @NotEmpty
    @Length(max = 20)
    public String getFirstName() { return firstName; }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

    @NotEmpty
    @Length(max = 20)
    public String getLastName() { return lastName; }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    @Valid
    public Address getAddress() { return address; }

    public void setAddress(Address address) {
        this.address = address;
    }

    @NotEmpty
    @Email
    public String getEmail() { return email; }

    public void setEmail(String email) {
        this.email = email;
    }

    @NotEmpty
    @Length(max = 20)
    public String getPassword() { return password; }

    public void setPassword(String password) {
        this.password = password;
    }
}</pre>

Let's talk a little about what's going on with the code above. As we noted earlier, we have a bean class (here, a class that represents user accounts of some sort) and we're using annotations to specify validation constraints. For a full list of the built-in constraints, see the <a href="http://www.hibernate.org/hib_docs/validator/reference/en/html/validator-defineconstraints.html#validator-defineconstraints-builtin">official Hibernate Validator documentation</a>, but we'll just focus on a small handful for right now.

First notice that we've attached our annotations to the getter methods. This is how we specify the validation constraints that attach to the associated properties.

We've used <code>@NotEmpty</code> in several places. This annotation means that the annotated property can't be null and it can't be the empty string either. There's also a <code>@NotNull</code> annotation that we could have used if we'd wanted to, but in this case I wanted to prevent the empty string from being used as values.

Another annotation that appears multiple times is the <code>@Length</code> annotation. We can specify associated minimum and maximum lengths by using the <code>min</code> and <code>max</code> elements, respectively, though in the example above we've specified only maximum lengths. (In the example that follows we'll see how to specify a minimum length as well.) So for example we've specified that passwords can't be any longer than 20 characters.

A third annotation is the <code>@Email</code> annotation. As you would guess, this indicates that the property must contain a valid e-mail address.

The fourth and final annotation of the ones that appear above is the <code>@Valid</code> annotation. This tells Hibernate Validator that it should validate the associated object&mdash;here an associated <code>Address</code> object&mdash;using whatever validation annotations we define on the <code>Address</code> class.

And that's our segue into our second example, the <code>Address</code> bean class, which appears below.

<pre>package com.wheelersoftware.demos.hibernatevalidator;

import org.hibernate.validator.Length;
import org.hibernate.validator.NotEmpty;
import org.hibernate.validator.Pattern;

public class Address {
    private String street1;
    private String street2;
    private String city;
    private String state;
    private String zip;

    @NotEmpty
    @Length(max = 40)
    public String getStreet1() { return street1; }

    public void setStreet1(String street1) {
        this.street1 = street1;
    }

    // No validation constraints
    public String getStreet2() { return street2; }

    public void setStreet2(String street2) {
        this.street2 = street2;
    }

    @NotEmpty
    @Length(max = 40)
    public String getCity() { return city; }

    public void setCity(String city) {
        this.city = city;
    }

    @NotEmpty
    @Length(max = 3)
    public String getState() { return state; }

    public void setState(String state) {
        this.state = state;
    }

    @NotEmpty
    @Length(min = 5, max = 5, message = "{zip.length}")
    @Pattern(regex = "[0-9]+")
    public String getZip() { return zip; }

    public void setZip(String zip) {
        this.zip = zip;
    }
}</pre>

Our validation annotations for <code>Address</code> are pretty similar to what we saw for <code>User</code>, but there are a few differences worth mentioning. First, notice that we don't have to attach validation constraints to every property. It's OK, for example, for <code>street2</code> to be null or anything else, so we simply refrain from defining validation constraints for this property.

Second, we're using a <code>@Pattern</code> annotation for the <code>zip</code> property. This allows us to specify regular expression match patterns.

The third and final difference is the <code>@Length</code> annotation we've defined for the <code>zip</code> property. Besides including a <code>min</code> element (which, when combined with the <code>max</code> element, indicates that the ZIP code must be exactly five characters long), we've also included a <code>message</code> element. We can use this element to do either of two things. First, we can use it to define a hardcoded message to display when the given validation constraint fails. That's not what we're doing here. Instead we're doing the second thing we can do, which is specify a message key using the brace syntax: <code>message = {key_name}</code>. The idea is that we'll eventually use this message key to look up a message in a resource bundle, thus externalizing the message. Later in the tutorial we'll map the <code>zip.length</code> message key to an actual message using a resource bundle.

That's it for the validation constraints themselves. Now let's see how we can tell Hibernate Validator to use them to perform our bean validation.

<h3>How to tell Hibernate Validator to validate annotated beans</h3>

It's fairly straightforward to get Hibernate Validator to validate our annotated beans. The following listing presents some demo code for doing exactly this. Let's take a look.

<pre>package com.wheelersoftware.demos.hibernatevalidator;

import org.hibernate.validator.ClassValidator;
import org.hibernate.validator.InvalidValue;

public final class Demo {
    private static ClassValidator&lt;User&gt;
        userValidator = new ClassValidator&lt;User&gt;(User.class);

    public static void main(String[] args) {
        validateUser(createUser());
    }

    private static User createUser() {
        User user = new User();
        user.setFirstName("123456789012345678901");
        user.setEmail("aol.com");

        Address addr = new Address();
        addr.setStreet1("");
        addr.setCity("Moreno Valley");
        addr.setState("CA");
        addr.setZip("QWERTY");
        user.setAddress(addr);

        return user;
    }

    private static void validateUser(User user) {
        InvalidValue[] invalidValues = userValidator.getInvalidValues(user);
        for (InvalidValue value : invalidValues) {
            System.out.println("========");
            System.out.println(value);
            System.out.println("message=" + value.getMessage());
            System.out.println("propertyName=" + value.getPropertyName());
            System.out.println("propertyPath=" + value.getPropertyPath());
            System.out.println("value=" + value.getValue());
        }
    }
}</pre>

There are really only a few interesting things happening here. First, we create a <code>ClassValidator&lt;User&gt;</code> instance for validating our <code>User</code> beans. We have to associate the <code>ClassValidator</code> with a type (here, <code>User</code>) because this is where Hibernate Validator goes in and reads all the annotations off of the bean class in question.

Next, we have to create the bean we want to validate. Usually this would come from a user form (for example, a web-based form, or maybe a Swing-based form) though that's not necessarily the case. Here we just create a bean manually, and we intentionally make a lot of the fields invalid so we can see how Hibernate Validator responds.

Finally we make the call <code>userValidator.getInvalidValues(user)</code>. This generates an array of <code>InvalidValue</code> instances&mdash;one for each validation constraint violation. Let's examine the <code>InvalidValue</code> class in more detail.

<h3>Understanding InvalidValue</h3>

To understand <code>InvalidValue</code> it will help to run the demo. So do that right now. You should see output that looks something like this:

<pre>26 [main] INFO org.hibernate.validator.Version - Hibernate Validator 3.1.0.GA
45 [main] INFO org.hibernate.annotations.common.Version - Hibernate Commons Annotations 3.1.0.GA
========
username may not be null or empty
message=may not be null or empty
propertyName=username
propertyPath=username
value=null
========
firstName length must be between 0 and 20
message=length must be between 0 and 20
propertyName=firstName
propertyPath=firstName
value=123456789012345678901
========
lastName may not be null or empty
message=may not be null or empty
propertyName=lastName
propertyPath=lastName
value=null
========
email not a well-formed email address
message=not a well-formed email address
propertyName=email
propertyPath=email
value=aol.com
========
password may not be null or empty
message=may not be null or empty
propertyName=password
propertyPath=password
value=null
========
street1 may not be null or empty
message=may not be null or empty
propertyName=street1
propertyPath=address.street1
value=
========
zip {zip.length}
message={zip.length}
propertyName=zip
propertyPath=address.zip
value=QWERTY
========
zip must match "[0-9]+"
message=must match "[0-9]+"
propertyName=zip
propertyPath=address.zip
value=QWERTY</pre>

If you look at the source code for <code>Demo.validateUser()</code> you'll see that we're printing out each <code>InvalidValue</code> instance itself (the first line) as well as the values of the <code>message</code>, <code>propertyName</code>, <code>propertyPath</code> and <code>value</code> properties (the rest of the lines). When we print out the instance itself, we get Hibernate Validator's attempt at a user-friendly validation eror message. It starts with the property name and then appends the message; examples include

<ul class="square">
<li><code>email not a well-formed e-mail address</code></li>
<li><code>password may not be null or empty</code></li>
<li><code>zip must match "[0-9]+"</code></li>
</ul>

Each <code>message</code> value (such as <code>may not be null or empty</code>) is a default associated with a validation annotation (such as <code>@NotEmpty</code>). Note that in the case of the length violation for the <code>zip</code> property, we're seeing not the default length message, but instead the new message key name (namely, <code>zip.length</code>) we specified in listing 2. We'll see how to map <code>zip.length</code> to an actual message using a resource bundle in a little bit. For now we're seeing just the key name because we haven't yet associated a message with the key.

The <code>propertyName</code> property specifies the name of the bean property whose value is invalid. For example, if we provide a bad ZIP code, then <code>propertyName</code> is <code>zip</code>.

The <code>propertyPath</code> property is similar to <code>propertyName</code> property, except that with <code>propertyPath</code> we get to see the path from the top-level bean down to the invalid property. You can see the difference, for instance, with the invalid ZIP code: <code>propertyName</code> is <code>zip</code> but <code>propertyValue</code> is <code>address.zip</code>.

Finally, <code>value</code> is just the bad value that violated the validation constraint in the first place.

While we can use <code>InvalidValue</code> itself as a source of semi-user-friendly messages, it's clear that the defaults leave something to be desired. After all, <code>zip must match "[0-9]+"</code> wouldn't be what most users would consider user-friendly. We can however provide a resource bundle to improve the error messages and that's what we'll look at now.

<h3>Improving validation error messages using ValidatorMessages.properties</h3>

With Hibernate Validator we can override the default messages associated with the various validation annotations. We can also provide highly specific error messages associated with property/constraint pairs. We do this using resource bundles. Though it's possible to provide Hibernate Validator with arbitrary resource bundles, the easiest approach is to create a <code>ValidatorMessages.properties</code> resource bundle on the classpath and use that. Hibernate Validator knows to look for that particular bundle (including any associated localizations) and use it as a message source. The next listing presents a simple example.

<pre>validator.notEmpty=may not be null or empty!
validator.length=must be {max} or fewer characters.
zip.length=Please enter a {max}-character ZIP code.</pre>

The first two lines of listing 3 provide alternatives for the <code>validator.notEmpty</code> and <code>validator.length</code> message keys. The key names are defined by the validation annotations themselves, so consult the Javadocs for the annotations if you need the key names (though you should be able to figure them out using the examples above).

Notice the <code>{max}</code> that appears in the message for the <code>validator.length</code> key. You can reference annotation elements from messages using the brace syntax. Consult the Javadocs for the various annotations&mdash;or else the Hibernate Validator reference manual&mdash;for a complete list of annotations and annotation elements.

The third line also specifies a message, but this time we're associating a message with the <code>zip.length</code> custom key we defined in listing 2. Note that here we don't use the braces for the key name. And also note that we can still reference annotation elements using the brace syntax. The custom message key approach is useful when you want to be very specific about the error message you provide to the end user.

<p>Here's what it looks like when you run it:</p>

<pre>29 [main] INFO org.hibernate.validator.Version - Hibernate Validator 3.1.0.GA
59 [main] INFO org.hibernate.annotations.common.Version - Hibernate Commons Annotations 3.1.0.GA
========
username may not be null or empty!
message=may not be null or empty!
propertyName=username
propertyPath=username
value=null
========
firstName must be 20 or fewer characters.
message=must be 20 or fewer characters.
propertyName=firstName
propertyPath=firstName
value=123456789012345678901
========
lastName may not be null or empty!
message=may not be null or empty!
propertyName=lastName
propertyPath=lastName
value=null
========
email not a well-formed email address
message=not a well-formed email address
propertyName=email
propertyPath=email
value=aol.com
========
password may not be null or empty!
message=may not be null or empty!
propertyName=password
propertyPath=password
value=null
========
street1 may not be null or empty!
message=may not be null or empty!
propertyName=street1
propertyPath=address.street1
value=
========
zip Please enter a 5-character ZIP code.
message=Please enter a 5-character ZIP code.
propertyName=zip
propertyPath=address.zip
value=QWERTY
========
zip must match "[0-9]+"
message=must match "[0-9]+"
propertyName=zip
propertyPath=address.zip
value=QWERTY</pre>

<h3>Discussion</h3>

That wraps it up for this tutorial. It's a good idea to get familiar with Hibernate Validator for a number of reasons:

<ul class="square">
<li>Annotation-based declarative annotations are an intuitive and convenient way to specify validation constraints.</li>
<li>Hibernate Validator isn't tied to any particular application tier.</li>
<li>It will support the JSR 303 standard once that finalizes.</li>
</ul>

We haven't explored everything that Hibernate Validator has to offer. For instance, it's possible to configure it such that Hibernate ORM automatically runs Hibernate Validator when performing persistence operations.

One slight annoyance that I've found is that there doesn't seem to be built-in way to substitute the bad value into the message itself. Sometimes I like to have messages like

<code>willie2gmail.com is not a valid e-mail address</code>

I suppose that I could create messages like

<code>{1} is not a valid e-mail address</code>

and then perform the substitution when processing the <code>InvalidValue</code> array, but it would be nice to be able to do this out of the box. Maybe there's a way to do it after all, but if so, I haven't found it.

Also, the documentation for Hibernate Validator is a little thin, both with respect to the reference manual and the Javadocs. Hopefully as JSR 303 matures we'll see improvements in this area. In the meantime, this tutorial will be my contribution to helping people understand Hibernate Validator.

<div class="endnote">Post migrated from my Wheeler Software site.</div>
