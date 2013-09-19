---
link: http://springinpractice.com/2008/11/25/publish-an-rss-feed-with-spring-3-0/
layout: post
title: Publish an RSS feed with Spring 3.0
date: 2008-11-25 14:27:00
comments: true
categories: [Chapter 08 - Communicating, Tutorials]
---
At the time of this writing, Spring 3.0 isn't out yet, but that doesn't mean we can't start goofing around with the <a href="http://static.springframework.org/downloads/nightly/snapshot-download.php?project=SPR">nightly snapshots</a>. I've been doing just that, and in this article I'll show you how to publish an RSS feed using the new <code>AbstractRssFeedView</code> class from Spring 3.0.

Those familiar with <a href="https://springmodules.dev.java.net/">Spring Modules</a> might recognize our view class. It began life as <code>AbstractRssView</code> in the Spring Modules project. But as of Spring 3.0, it's now a first-class member of the framework (though it's been renamed to <code>AbstractRssFeedView</code>), along with the <code>AbstractAtomFeedView</code> class for Atom feeds, and the <code>AbstractFeedView</code>, which serves as a base class for both. The new classes, like the old one, are based on Sun's <a href="https://rome.dev.java.net/">ROME API</a>.

You'll need to know Spring Web MVC to get the most out of this article. In particular, you'll need to understand the relationship between controllers and views, which in a nutshell is this: when a controller is done processing a request, it returns a logical view name that a view resolver subsequently maps to a view.

Without further ado, let's look at some code.

<h3>The controller</h3>

Let's start with the controller, since processing starts there, and since that's probably more familiar to more readers than implementing views. Here's a pretty basic controller for a news feed.

<pre>package rssdemo.web;

import javax.servlet.http.HttpServletResponse;
import org.springframework.beans.factory.annotation.Required;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import rssdemo.model.NewsItem;
import rssdemo.service.NewsService;

@Controller
public final class NewsController {
    private NewsService newsService;
    private String rssNewsFeedView;
    
    @Required
    public void setNewsService(NewsService newsService) {
        this.newsService = newsService;
    }
    
    @Required
    public void setRssNewsFeedView(String rssNewsFeedView) {
        this.rssNewsFeedView = rssNewsFeedView;
    }
    
    @RequestMapping("/news.rss")
    public String rss(HttpServletResponse res, Model model) {
        List&lt;NewsItem&gt; newsItems = newsService.getAllNewsItems();
        model.addAttribute(newsItems);                                     // 1
        return rssNewsFeedView;                                            // 2
    }
}
</pre>

As you can see, this controller really is as simple as I said it would be. Our <code>rss()</code> method grabs a list of <code>NewsItem</code>s from the <code>NewsService</code> <span class="cueball">1</span> and places it on the <code>Model</code> under a conventionally-generated key, which in this case would be <code>newsItemList</code>. Then we simply return an injected logical view name <span class="cueball">2</span>, which we're going to map to an actual view shortly. (We inject the view name to maintain the separation between the controller and the view.)

Now let's check out the star of the show, which would be our view.

<h3>The view, via Spring 3.0's new AbstractRssFeedView</h3>

Now we come to the meat of the subject. Here we're going to implement a Spring Web MVC view by extending Spring 3.0's new <code>AbstractRssFeedView</code> class and by using ROME to model our news feed. Once we have the view in place, we'll be ready to configure the mapping from the logical view name we set in the controller to the view.

The next listing what's involved in implementing a view for an RSS feed. If you're doing an Atom feed, the approach is entirely analogous, but you would just need to extend <code>AbstractAtomFeedView</code> instead of <code>AbstractRssFeedView</code>.

<pre>package rssdemo.web;

import java.util.*;
import javax.servlet.http.*;
import org.springframework.beans.factory.annotation.Required;
import org.springframework.web.servlet.view.feed.AbstractRssFeedView;
import com.sun.syndication.feed.rss.Channel;
import com.sun.syndication.feed.rss.Description;
import com.sun.syndication.feed.rss.Item;
import rssdemo.model.NewsItem;

public final class RssNewsFeedView extends AbstractRssFeedView {           // 1
    private String feedTitle;                                              // 2
    private String feedDesc;
    private String feedLink;
    
    @Required
    public void setFeedTitle(String feedTitle) {
        this.feedTitle = feedTitle;
    }
    
    @Required
    public void setFeedDescription(String feedDesc) {
        this.feedDesc = feedDesc;
    }
    
    @Required
    public void setFeedLink(String feedLink) {
        this.feedLink = feedLink;
    }

    @Override
    protected void buildFeedMetadata(
            Map model, Channel feed, HttpServletRequest request) {         // 3
        
        feed.setTitle(feedTitle);
        feed.setDescription(feedDesc);
        feed.setLink(feedLink);
    }
    
    @Override
    protected List&lt;Item&gt; buildFeedItems(
            Map model,
            HttpServletRequest request,
            HttpServletResponse response)
        throws Exception {                                                 // 4
        
        @SuppressWarnings("unchecked")
        List&lt;NewsItem&gt; newsItems =
            (List&lt;NewsItem&gt;) model.get("newsItemList");                    // 5
        
        List&lt;Item&gt; feedItems = new ArrayList&lt;Item&gt;();
        for (NewsItem newsItem : newsItems) {                              // 6
            Item feedItem = new Item();
            feedItem.setTitle(newsItem.getTitle());
            feedItem.setAuthor(newsItem.getAuthor());
            feedItem.setPubDate(newsItem.getDatePublished());
            
            Description desc = new Description();
            desc.setType("text/html");
            desc.setValue(newsItem.getDescription());
            feedItem.setDescription(desc);
            
            feedItem.setLink(newsItem.getLink());
            feedItems.add(feedItem);
        }
        
        return feedItems;
    }
}</pre>

So what's going on here? Well, we begin by extending the <code>AbstractRssFeedView</code> class <span class="cueball">1</span>. Then we include some injected fields for feed metadata <span class="cueball">2</span>. We use these in the optional <code>buildFeedMetadata()</code> method <span class="cueball">3</span>. I say "optional" because the <code>AbstractRssFeedView</code> provides a dummy implementation (actually, its superclass <code>AbstractFeedView</code> provides it).

The method we're required to implement is <code>buildFeedItems()</code> <span class="cueball">4</span>. Here's where we map our list of domain objects (here, a list of <code>NewsItem</code> objects that are just something we cooked up ourselves) to the ROME API, which provides a structure for modeling feeds. We begin by grabbing the <code>NewsItem</code> list off the model (recall from listing 1 that we placed the list on the model in the controller) <span class="cueball">5</span>. Then we iterate over the <code>NewsItem</code> list, mapping each one to a ROME <code>Item</code> <span class="cueball">6</span>.

That should be pretty straightforward-looking. And we're almost done. There's just one part that remains, and that's establishing the mapping between the logical view name we return from the controller and the <code>RssNewsFeedView</code> that we just created. We do that in the application context.

<h3>Configure the view resolver for your RSS feed</h3>

The last thing we need to take care of is configuring up the right view resolver in our application context. We can't use the standard <code>InternalResourceViewResolver</code> here because we're not mapping to a URL; instead we want to map to a view (namely, the <code>RssNewsFeedView</code> that we just created) and we want that view to handle generating the output directly.

The simplest approach here is to use something called the <code>BeanNameViewResolver</code>. The idea with this type of resolver is quite simple. Whenever a controller returns a logical view name, the <code>BeanNameViewResolver</code> will attempt to find a bean on the application context with the same name (or ID). If there's a match, then it interprets that bean as the mapped view. Otherwise, the other resolvers are given their shot at matching the view name.

To add a <code>BeanNameViewResolver</code>, all we need to add is this:

<pre>&lt;bean class="org.springframework.web.servlet.view.BeanNameViewResolver"/&gt;</pre>

We also need to inject the view name into our controller, as we saw in listing 1. Here's how we can do that:

<pre>&lt;bean class="rssdemo.web.NewsController"
    p:newsService-ref="newsService"
    p:rssNewsFeedView="rssNewsFeedView"/&gt;</pre>

We just chose the name <code>rssNewsFeedView</code> more or less arbitrarily; we could have chosen anything. It's good to choose something accurate and descriptive just to keep things clear and the minimize the chance of a mapping conflict, since the name you choose is going to be a bean name. Speaking of which, we'll need to put our view on the app context too:

<pre>&lt;bean id="rssNewsFeedView"
    class="rssdemo.web.RssNewsFeedView"
    p:feedTitle="Ye Olde Cigar Shoppe"
    p:feedDescription="Latest and greatest news about cigars"
    p:feedLink="http://yeoldecigarshoppe.com/"/&gt;</pre>

And there you have it! When the <code>DispatcherServlet</code> gets a request for the RSS feed, it will run the controller method and then grab the resulting view name, which we've configured to be <code>rssNewsFeedView</code>. Then the <code>BeanNameViewResolver</code> will find the corresponding view bean on the app context&mdash;in this case our <code>RssNewsFeedView</code>&mdash;and the view bean will finish up the request as required. All in all a nice, clean way of handling feed publication.

<span class="icon stickyNote">Post migrated from my Wheeler Software site.</span>