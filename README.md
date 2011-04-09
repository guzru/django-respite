# Respite

## About

Respite conforms Django to [Representational State Transfer (REST)](http://en.wikipedia.org/wiki/Representational_State_Transfer).
## Requirements

* Django v1.3 or later

## Usage

Respite is influenced by Ruby on Rails, though in the spirit of Python it is not nearly as "magic". It will, however, save you a lot of code:

    # models.py
    
    from django import models
    
    class Article(models.Model):
        title = models.CharField(max_length=255)
        content = models.TextField()
        published = True
        created_at = models.DateTimeField(auto_now_add=True)


    # urls.py
    
    from django.conf.urls.defaults import *
    from respite.urls import resource
    from views import ArticleView
    
    urlpatterns = resource(
        prefix = 'articles',
        view = ArticleView
    )


    # views.py
    
    from respite import View
    from models import Article
    
    class ArticleView(View):
        model = Article
        template_path = 'articles'
        supported_formats = ['html', 'json']
    
    # templates/articles/index.html
    
    <!DOCTYPE html>
    <html>
        <head>
            <title>{{ article.title }}</title>
        </head>
        <body>
            {% for article in articles %}
            <article>
                <h1><a href="{% url article id=article.id %}">{{ article.title }}</a></h1>
                <time datetime="{{ article.created_at.isoformat }}">{{ article.created_at }}</time>
                <p>
                    {{ article.content }}
                </p>
            </article>
            {% endfor %}
        </body>
    </html>
    
    # templates/articles/index.json
    
    [
        {% for article in articles %}
        {
            "id": article.id,
            "title": "{{ article.title }}",
            "content": "{{ article.content }}",
            "published": {{ article.published|lower }}
            "created_at": "{{ article.created_at.isoformat }}",
            "url": "{% url article id=article.id %}"
        }{% if not forloop.last %},{% endif %}
        {% endfor %}
    ]
    
    ...

Respite's `View` class defines a collection of functions for viewing and manipulating model instances;
`index`, `show`, `new`, `create`, `edit`‚ `update` and `destroy`.

    HTTP method         HTTP path           Function            Purpose
    
    GET                 articles/           index               Render a list of articles
    GET                 articles/new        new                 Render a form to create a new article
    POST                articles/           create              Create a new article
    GET                 articles/1          show                Render a specific article
    GET                 articles/1/edit     edit                Render a form to edit a specific article
    PUT                 articles/1          update              Edit a specific article
    DELETE              articles/1          destroy             Delete a specific article
    
In a nutshell, Respite provides you with a collection of features you probably need for most of your models and routes them
RESTfully. You can override any or all of these functions and customize them as you'd like. For example, you could only list
articles that have been published:

    class ArticleView(View):
        model = Article
        template_path = 'articles'
        supported_formats = ['html', 'json']
        
        def index(self, request):
            articles = self.model.objects.filter(published=True)
            
            return self._render(
                request = request,
                template = 'index',
                context = {
                    'articles': articles,
                },
                status = 200
            )


## Installation

* `pip install git+http://github.com/jgorset/respite.git`
* Add `respite.middleware.HTTPPUTMiddleware` your middleware classes.

If you're not just building an API, you might also want to add `respite.middleware.HTTPMethodOverrideMiddleware`
to your middleware classes; it facilitates for overriding the HTTP method with the `X-HTTP-Method-Override` header or a
`_method` HTTP POST parameter, which is the only way to update (HTTP PUT) and delete (HTTP DELETE) resources from
a web browser.

