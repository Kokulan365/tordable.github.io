---
layout: post
title: A small blogging platform in Google App Engine
highlighter: rouge
---


If you have never made a
<a href="http://en.wikipedia.org/wiki/Web_application">web application</a>
it may seem daunting. There are hundreds of alternative technologies and
frameworks out there. And web apps development is quite different from client
applications, which is what most developers are used to.


<p>
Here is an example of a web application. Wikipedia!
</p>
<img src="/images/screenshot-wikipedia.png" alt="Screenshot of Wikipedia" />
<p>
Most web applications share a few common elements:
</p>
<ul>
<li>A persistence layer, for authored content or user created content</li>
<li>A system to connect each user request with a part of the application</li>
<li>A method to render and display that content to users</li>
</ul>
<p>
Traditionally the persistence layer is a
<a href="http://en.wikipedia.org/wiki/SQL_database">SQL database</a>,
the requests are directed for example to
<a href="http://en.wikipedia.org/wiki/Servlet">Java servlets</a> in an app
server like <a href="http://tomcat.apache.org/">Tomcat</a>, and there is a
more or less refined templating engine, in which the content is added to
create the whole page returned to the user.
</p>
<p>
Another alternative is
<a href="http://code.google.com/appengine/">Google App Engine</a>.
Google App Engine is a platform and a set of libraries to develop web
applications based in Google's own infrastructure. It is available in
Java and Python, but here I will concentrate on the Python version.
</p>
<p>
The persistence layer of Google App Engine is the
<a href="http://code.google.com/appengine/docs/python/datastore/overview.html">
  Datastore</a>, a highly parallel but simple to use storage solution.
The Datastore doesn't support queries as complex SQL does, however it can
scale up to a level which is beyond what a normal database can do. And it
can do so in a way that is trivial for the developer.
</p>

<p>
Google App Engine uses <a href="http://en.wikipedia.org/wiki/YAML">yaml</a>
and the webapp framework to answer user queries. One can set a configuration
file which assigns certain url paths (via a regular expression) to instances
of webapp.RequestHandler.
</p>

<pre>
handlers:
- url: /.*
  script: app.py
</pre>

<p>
The instance can implement a get() method which generates the response
returned to the user.
</p>

``` python
class App(webapp.RequestHandler):
    def get(self):
        self.response.headers['Content-Type'] = 'text/plain'
        self.response.out.write('Hello, World!')
```

<p>
Finally, in order to generate HTML, Google App Engine incorporates the
templating engine from <a href="http://www.djangoproject.com/">Django</a>.
A template is essentially a document with <em>variables</em>
instead of content. When the application needs to answer a user request it can
load the template and replace the <em>variables</em> with real content,
for example information from the data storage.
</p>
<p>
This is basically the way that this blog is made. It is a very simple Google
App Engine application. It uses the Datastore for the blog posts, which are
retrieved when it receives a request for the /blog url path. Then it replaces
the blog post content into the blog template and returns that content.
Here is a sample of code. It is not the actual code, but it gives a complete
example:
</p>

``` python
class Blog(webapp.RequestHandler):
  def get(self):
    # Retrieve the posts from the database.
    query = 'SELECT * FROM Post WHERE public = True ORDER BY date DESC '
         'LIMIT %d ' % NUM_POSTS_IN_MAIN_PAGE
    posts = db.GqlQuery(query).fetch(NUM_POSTS_IN_MAIN_PAGE)

    # Render them into html.
    if len(posts) > 0:
      template_values = &#123;'posts': posts &#125;
      template_path = os.path.join(os.path.dirname(__file__), 'templates/blog')
      content = template.render(template_path, template_values)
    else:
      content = ''

    # And return a full page with the blog content.
    s = pages.FullPageGenerator()
    self.response.out.write(s.generate(content))
```

<p>
To sum up, Google App Engine is a great option for developing web applications.
Expecially if you will require it to scale seamlessly, or to integrate with
Google services. Check out the
<a href="http://code.google.com/appengine/docs/python/gettingstarted/introduction.html">
  Google App Engine tutorial</a>.
</p>
<img src="http://code.google.com/appengine/images/appengine-noborder-120x30.gif"
alt="Powered by Google App Engine" />
