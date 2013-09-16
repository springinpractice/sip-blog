---
layout: post
title: "How to run JavaScript from Java"
date: 2012-05-13 11:02
comments: true
categories: [Chapter 09 - Comments, Quick Tips]
---
Java 6 comes with the [Rhino JavaScript engine](http://www.mozilla.org/rhino/), which makes it easy to run JavaScript from inside your Java app. There are different situations in which you might want to do this. Chapter 9 of [Spring in Practice](http://www.manning.com/wheeler/) affords a good example. There we're implementing a rich-text comment engine based on the WMD editor that [Stack Overflow](http://stackoverflow.com) uses. We have a <code>showdown.js</code> script that maps Markdown to HTML, and we want to run it in two places:

* on the client to present a preview pane
* on the server to store the Markdown as HTML for easy presentation (as opposed to running the script against every comment dynamically)

Here's how we can run it on the server:

    import javax.script.ScriptEngine;
    import javax.script.ScriptEngineManager;
    import javax.script.ScriptException;
    
    ...
    
    public final class RichTextFilter implements TextFilter {
        @Inject private File showdownJsFile;
    
        private String markdownToHtml(String markdown) {
            try {
                ScriptEngineManager mgr = new ScriptEngineManager();
                ScriptEngine engine = mgr.getEngineByName("JavaScript");
                engine.eval(showdownJs);
                engine.eval("var markdown = '" + markdown + "';");
                engine.eval("var converter = new Showdown.converter();");
                engine.eval("var html = converter.makeHtml(markdown);");
                return (String) engine.get("html");
            } catch (ScriptException e) {
                // Shouldn't happen unless somebody breaks the script
                throw new RuntimeException(e);
            }
        }
    
        ... other stuff ...
    }

To see how to inject the File into the class in Spring, see my post [Injecting a file from the classpath into a bean](http://springinpractice.com/2012/05/12/injecting-a-file-from-the-classpath-into-a-bean/).
