---
layout: post
title: "ch01  building a blog"
description: ""
category: django
tags: [django,coding,]
---
{% include JB/setup %}

Creating an isolated Python environment

# pip install virtualenv
# virtualenv my_env

conda 환경에서는 virtualenv 를 사용하지 않고, “conda create” 명령어를 사용한다.
conda에서 가상환경을 생성하면, “anaconda/envs/[yourenvname]  하부에 폴더구조가 생성된다.

# conda create -n[ yourenvname] python=x.x anaconda
생성한 가상 환경을 활성화 시킬 때는,
# source activate [yourenvname]
가상환경에 파이썬 패키지를 설치할 때는,
# conda install -n [yourenvname] [package]
가상환경을 비활성화 시킬 때는
# source deactivate
가상환경을 삭제할 때는
# conda remove -n [yourenvname] -all

1) Installing Django with pip

# pip install Django

(py35)swjo207ui-MacBook-Pro-4:~ swjo207$ python
Python 3.5.1 |Continuum Analytics, Inc.| (default, Dec  7 2015, 11:24:55)
[GCC 4.2.1 (Apple Inc. build 5577)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import django
>>> django.VERSION
(1, 9, 4, 'final', 0)

2) Creating your  first project
# django-admin startproject mysite
# cd mysite
# python manage.py migrate

2-1) Running the development server
# python manage.py runserver
# python manage.py runserver 127.0.0.1:8001 \
   --settings=mysite.settings

You can  find more information about how to deploy Django with different web servers at https://docs. djangoproject.com/en/1.8/howto/deployment/wsgi/.

2-2) Project settings
open [settings.py]
You can see all settings and their default values in https://docs.djangoproject.com/en/1.8/ref/settings/.

2-3) Projects and applications

Throughout this book, you will read the terms project and application over and over. In Django, a project is considered a Django installation with some settings; and an application is a group of models, views, templates, and URLs. Applications interact with the framework to provide some speci c functionalities and may be reused in various projects. You can think of the project as your website, which contains several applications like blog, wiki, or forum, which can be used in other projects.

2-4) Creating an application
# python manage.py startapp blog

3) Designing the blog data schema

blog/models.py

from django.db import models
from django.utils import timezone
from django.contrib.auth.models import User

class Post(models.Model):
    STATUS_CHOICES = (
        ('draft', 'Draft'),
        ('published', 'Published'),
    )
    title = models.CharField(max_length=250)
    slug = models.SlugField(max_length=250, unique_for_date='published')
    author = models.ForeignKey(User,related_name='blog_posts')
    body = models.TextField()
    published = models.DateTimeField(default=timezone.now)
    created = models.DateTimeField(auto_now_add=True)
    updated = models.DateTimeField(auto_now=True)
    status = models.CharField(max_length=10,choices=STATUS_CHOICES,default='draft')

    class Meta:
        ordering = ('-published',)

    def __str__(self):
        return self.title

You can find all field types in https://docs.djangoproject.com/ en/1.8/ref/models/fields/.

3-1) Activating your application
mysite/settings.py

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'blog',
]

3-2) Creating and applying migrations
# python manage.py makemigrations blog
위 명령어가 수행하는 실제 SQL 질의어를 살펴보기 위해서
# python manage.py sqlmigrate blog 0001 를 실행해 본다.

BEGIN;
--
-- Create model Post
--
CREATE TABLE "blog_post" ("id" integer NOT NULL PRIMARY KEY AUTOINCREMENT, "title" varchar(250) NOT NULL, "slug" varchar(250) NOT NULL, "body" text NOT NULL, "published" datetime NOT NULL, "created" datetime NOT NULL, "updated" datetime NOT NULL, "status" varchar(10) NOT NULL, "author_id" integer NOT NULL REFERENCES "auth_user" ("id"));
CREATE INDEX "blog_post_2dbcba41" ON "blog_post" ("slug");
CREATE INDEX "blog_post_4f331e2f" ON "blog_post" ("author_id");

COMMIT;

Let's sync our database with the new model. Run the following command to apply existing migrations:
# python manage.py migrate

4) Creating an administration site for your models

Now that we have de ned the Post model, we will create a simple administration site to manage blog posts. Django comes with a built-in administration interface that is very useful for editing content. The Django admin site is built dynamically by reading your model metadata and providing a production-ready interface for editing content. You can use it out-of-the-box, con guring how you want your models to be displayed in it.

4-1) Creating a superuser
관리 페이지를 사용하기 위해 먼저 시스템 관리자 계정을 생성해야 한다.
# python manage.py createsuperuser

4-2) The Django administration site

Now, start the development server with the command python manage.py runserver and open http://127.0.0.1:8000/admin/ in your browser. You should see the administration login page.

4-3) Adding your models to the administration site
mysite/admin.py

from django.contrib import admin
from .models import Post

admin.site.register(Post)

관리페이지로 다시 들어가면, BLOG/Posts 메뉴가 새로 추가된 것을 확인할 수 있다.
[Add] 링크를 클릭하면, 자동으로 항목을 추가할 수 있는 폼이 나타나는 것을 확인해 볼 수 있으며 내용을 채워 넣고 Post 필드를 하나 추가할 수 있다. 이런 식으로 Post 테이블에 대해서 CRUD 를 각각 해 볼 수 있는 관리자 인터페이스가 새로 만들어 진 것을 확인해 본다.

4-4) Customizing the way models are displayed

Now we are going to see how to customize the admin site. Edit the admin.py le of your blog application and change it into this:

from django.contrib import admin
from .models import Post

class PostAdmin(admin.ModelAdmin):
    list_display = ('title', 'slug', 'author', 'publish','status')
    list_filter = ('status', 'created', 'publish', 'author')
    search_fields = ('title', 'body')
    prepopulated_fields = {'slug': ('title',)}
    raw_id_fields = ('author',)
    date_hierarchy = 'publish'
    ordering = ['status', 'publish']

admin.site.register(Post,PostAdmin)

list_display: 화면에 표시될 필드 리스트이다. 기본은 Post 였지만 위와 같이 정의한 5개의 필드가 화면에 표시됨
list_filter: 항목에 대해 필터링을 할 수 있다. 화면 우측에 표시됨
search_fields: 검색 폼으로 찾을 항목
date_hierarchy: 검색 폼 밑에 나타나는 네비게이션이 표시 됨
ordering: 항목을 정렬하는 기준이 되는 필드, 화면에 오름차순/내림차순으로 정렬할 수 있도록 표시 됨
prepopulated_fields: Add post 를 클릭하여, 추가화면이 보여질 때 ‘slug’ 필드가 제목 값으로 자동으로 채워진다.

5) Working with QuerySet and managers
Now that you have a fully functional administration site to manage your blog's content, it's time to learn how to retrieve information from the database and interact with it.

Django comes with a powerful database-abstraction API that lets you create, retrieve, update, and delete objects easily. The Django Object-relational Mapper (ORM) is compatible with MySQL, PostgreSQL, SQLite, and Oracle. Remember that you can de ne the database of your project by editing the DATABASES setting in the settings.py le of your project. Django can work with multiple databases at a time and you can even program database routers that handle the data in any way you like.
Once you have created your data models, Django gives you a free API to interact with them. You can  nd the data model reference of the of cial documentation at https://docs.djangoproject.com/en/1.8/ref/models/.

5-1) Creating objects
# python manage.py shell
 >>> from django.contrib.auth.models import User
 >>> from blog.models import Post
 >>> user = User.objects.get(username='admin')
 >>> Post.objects.create(title='One more post’,  slug='one-more-post’, body='Post body.’, author=user)
 >>> post.save()
Post.objects.create(title='One more post', slug='one-more-post',body='Post body.', author=user)
5-2) Updating objects
>>> post.title = 'New title'
>>> post.save()
5-3) Retrieving objects
>>> all_posts = Post.objects.all()
>>> Post.objects.all()
Using the  filter() method
>>> Post.objects.filter(publish__year=2015)
>>> Post.objects.filter(publish__year=2015, author__username='admin')
Using exclude()
>>> Post.objects.filter(publish__year=2015)\

        .exclude(title__startswith='Why')
Using order_by()
>>> Post.objects.order_by('title')
5-4) Deleting objects
>>> post = Post.objects.get(id=1)
>>> post.delete()

5-5) Creating model managers
blog/models.py

from django.db import models
from django.utils import timezone
from django.contrib.auth.models import User

class PublishedManager(models.Manager):
    def get_queryset(self):
        return super(PublishedManager,
            self).get_queryset() \
                .filter(status='published')

class Post(models.Model):
    STATUS_CHOICES = (
        ('draft', 'Draft'),
        ('published', 'Published'),
    )
    title = models.CharField(max_length=250)
    slug = models.SlugField(max_length=250, unique_for_date='publish')
    author = models.ForeignKey(User,related_name='blog_posts')
    body = models.TextField()
    publish = models.DateTimeField(default=timezone.now)
    created = models.DateTimeField(auto_now_add=True)
    updated = models.DateTimeField(auto_now=True)
    status = models.CharField(max_length=10,choices=STATUS_CHOICES,default='draft')

    objects = models.Manager() # default manager
    published = PublishedManager() # custom manager

    class Meta:
        ordering = ('-publish',)

    def __str__(self):
        return self.title

get_queryset() is the method that returns the QuerySet to be executed. We use it to include our custom  filter in the  final QuerySet. We have defined our custom manager and added it to the Post model; we can now use it to perform queries. For example, we can retrieve all published posts whose title starts with Who using:
 >>> Post.published.filter(title__startswith='Who')

6) Building list and detail views

6-1) Creating list and detail views

blog/views.py

from django.shortcuts import render
from .models import Post

def post_list(request):
    posts = Post.published.all()
    return render(request,
            'blog/post/list.html',
            {'posts': posts}
        )

def post_detail(request, year, month, day, post):
    post = get_object_or_404(Post, slug=post,
                                status='published',
                                publish__year=year,
                                publish__month=month,
                                publish__day=day
        )
    return render(request,
            'blog/post/detail.html',
            {'post': post}
        )

render():  shortcut provided by Django to render the list of posts with the given template. This function takes the request object as parameter, the template path and the variables to render the given template. It returns an HttpResponse object with the rendered text (normally HTML code). The render() shortcut takes the request context into account, so any variable set by template context processors is accessible by the given template. Template context processors are just callables that set variables into the context. You will learn how to use them in Chapter 3, Extending Your Blog Application.
post_detail: This view takes year, month, day, and post parameters to retrieve a published post with the given slug and date. Notice that when we created the Post model, we added the unique_for_date parameter to the slug eld. This way we ensure that there will be only one post with a slug for a given date, and thus, we can retrieve single posts by date and slug. In the detail view, we are using the get_object_or_404() shortcut to retrieve the desired Post. This function retrieves the object that matches with the given parameters, or launches an HTTP 404 (Not found) exception if no object is found. Finally, we use the render() shortcut to render the retrieved post using a template.

6-2) Adding URL patterns for your views

If you haven't worked with regular expressions before, you might want to take a look at https://docs.python.org/3/howto/regex.html first.
앞에서 만든 뷰 화면을 URL 요청에 연결시켜주는 설정을 해주어야 한다.
mysite 프로젝트 —> blog 앱 으로 연결되도록 urls.py 파일을 만들어 주고 계층적으로 설계를 하도록 한다.
blog앱의 urls.py 파일을 생성하고 아래와 같이 작성해주고, 프로젝트 urls.py 에 include 로 blog앱의 urls.py 를 추가한다.

blog/urls.py

from django.conf.urls import url
from . import views

urlpatterns = [
    #post views
    url(r'^$', views.post_list, name='post_list'),
    url(r'^(?P<year>\d{4})/(?P<month>\d{2})/(?P<day>\d{2})/', \
        r'(?P<post>[-\w]+)/$', views.post_detail, name='post_detail'),
]

mysite/urls.py

from django.conf.urls import url
from django.contrib import admin

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^blog/', include('blog.urls', namespace='blog', app_name='blog')),
]

6-3) Canonical URLs for models

You can use the post_detail URL that you have defined in the previous section to build the canonical URL for Post objects. The convention in Django is to add a get_absolute_url() method to the model that returns the canonical URL of the object. For this method, we will use the reverse() method that allows you to build URLs by their name and passing optional parameters. Edit your models.py le and add the following:

mysite/models.py

from django.core.urlresolvers import reverse
...
    def get_absolute_url(self):
        return reverse('blog:post_detail',
                args=[self.publish.year,
                    self.publish.strftime('%m'),
                    self.publish.strftime('%d'),
                    ])

7) Creating templates for your views

Create the following directories and  les inside your blog application directory:

templates/
    blog/


        base.html
        post/


            list.html
            detail.html

This will be the  le structure for our templates. The base.html le will include the main HTML structure of the website and divide the content into a main content area and a sidebar. The list.html and detail.html les will inherit from the base.html le to render the blog post list and detail views respectively.
Django has a powerful template language that allows you to specify how data is displayed. It is based on template tags, which look like {% tag %} template variables, which look like {{ variable }} and template filters, which can be applied to variables and look like {{ variable|filter }}. You can see all built-in template tags and  lters in https://docs.djangoproject.com/en/1.8/ref/templates/builtins/.

blog/templates/blog/base.html

{% load staticfiles %}
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %} {% endblock %}</title>
    <link rel="stylesheet" type="text/css" href="{% static "css/blog.css" %}">
</head>
<body>
    <div id="content">
        {% block content %}

        {% endblock content %}
    </div>
    <div id="sidebar">
        <h2> my blog </h2>
            <p> this is my blog </p>
    </div>
</body>
</html>

{% load staticfiles %}: load the staticfiles template tags that are provided by the django.contrib.staticfiles application. After loading it, you are able to use the {% static %} template  lter throughout this template.
{% block %}: we want to define a block in that area. Templates that inherit from this template can  fill the blocks with content. We have defined a block called title and a block called content.

blog/templates/blog/post/list.html

{% extends "blog/base.html" %}

{% block title %}
My blog
{% endblock title %}

{% block content %}
    <h1> my blog </h1>
    {% for  post in posts %}
        <h2>
            <a href="{{ post.get_absolute_url }}">    {{ post.title }} </a>
        </h2>
        <p class="date">
            published {{ post.publish }} by {{ post.author }}
        </p>
        {{ post.body|truncatewords:30|linebreaks }}
    {% endfor %}
{% endblock content %}

{% extends %}: inherit from the blog/base.html template
template filter: truncatewords truncates the value to the number of words speci ed, and linebreaks converts the output into HTML line breaks

8) Adding pagination
Django has a built-in pagination class that allows you to manage paginated data easily.
blog/views.py

from django.core.paginator import Paginator,EmptyPage, PageNotAnInteger
...
def post_list(request):
    object_list = Post.published.all()
    paginator = Paginator(object_list,3) # 3 post in each page
    page = request.GET.get('page')
    try:
        posts = paginator.page(page)
    except PageNotAnInteger:
        # 숫자가 아닌 경우 처리
        posts = paginator.page(1)
    except EmptyPage:
        # 아무것도 없는 경우
        posts = paginator.page(paginator.num_pages)
    return render(request,
            'blog/post/list.html',
            {'page': page,
            'posts': posts}
        )

blog/templates/blog/pagination.html

<div class="pagination">
    <span class="step-links">
        {% if page.has_previous %}
            <a href="?page={{ page.previous_page_number }}"> previous </a>
        {% endif %}
        <span class="current">
        page {{ page.number }} of {{ page.pagination.num_pages }}.
        </span>
        {% if page.has_next %}
            <a href="?page={{ page.next_page_number }}">next </a>
        {% endif %}
    </span>
</div>

blog/templates/blog/list.html

{% block content %}
    ...


    {% include "pagination.html" with page=posts %}


   {% endblock %}

9) Using class-based views

Since a view is a callable that takes a web request and returns a web response, you can also define your views as class methods. Django provides base view classes for this. All of them inherit from the View class, which handles HTTP method dispatching and other functionality. This is an alternate method to create your views.
We are going to change our post_list view into a class-based view to use the generic ListView offered by Django. This base view allows you to list objects of any kind.

blog/views.py

from django.views.generic import ListView
   class PostListView(ListView):
       queryset = Post.published.all()
       context_object_name = 'posts'
       paginate_by = 3
       template_name = 'blog/post/list.html'

blog/urls.py

urlpatterns = [
    # post views
    # url(r'^$', views.post_list, name='post_list'),
    url(r'^$', views.PostListView.as_view(), name='post_list'), url(r'^(?P<year>\d{4})/(?P<month>\d{2})/(?P<day>\d{2})/'\
           r'(?P<post>[-\w]+)/$',
           views.post_detail,
           name='post_detail'),
]

blog/templates/blog/post/list.html

{% include "blog/pagination.html" with page=page_obj %}
