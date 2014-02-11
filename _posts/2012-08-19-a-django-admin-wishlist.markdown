---
author: ankur
comments: true
date: 2012-08-19 09:28:32+00:00
layout: post
slug: a-django-admin-wishlist
title: A Django Admin Wishlist
wordpress_id: 18
categories:
- Django
- Programming
---



It is okay to skim this post and only read the parts that you find interesting.

When I was just learning how to use Django, I dismissed `django.contrib.admin` ("_the admin_" from now on) as a nice-to-have extra, a marginally useful demo of framework functionality, but not much more. I didn't even enable it for most of the apps I wrote as a Django novice. The simple, honest `django.forms.Form` and the darkly magical `django.forms.ModelForm` together did everything I needed. Learning to customize the admin was a waste of my time. Or so I thought.

As the scope of my projects grew, my attitude towards the admin thawed a little. I had heavily leveraged it for an app that I had written for my personal use and found that it was more customizable than I had initially believed. I was surprised to discover that I could make it _mostly_ work the way I wanted by making only minor adjustments to my Python code. While experimenting with some third-party Django apps, I found that all of them use the admin in one way or another. [Mezzanine](http://mezzanine.jupo.org/), [Cartridge](http://cartridge.jupo.org/), [django-cms](http://django-cms.org/), [Zinnia](http://django-blog-zinnia.com/), [Satchmo](http://www.satchmoproject.com/), [django-filebrowser](https://github.com/sehmaschine/django-filebrowser) ... all of them hook into and extend the admin framework instead of using something home-grown. Of course, sometimes the admin framework doesn't have the functionality these apps need, and the developers end up building the missing functionality from scratch, but it's the existing functionality that gets them 70-75% of the way.

Consequently, when I was starting work on an internal webapp for my company, I decided to not waste my time writing a custom management UI for it and instead use the admin from day one. This was about a month ago. Since then I have learned a lot about how to customize and extend the admin to my app's needs. I have also, in the course of customizing the admin, come across some annoying limitations in the admin framework. Some of these limitations are easily overcome using third-party apps, others require a bit of extra work on the developer's part, and still others need to be looked at by the core Django developers. This post details the limitations I ran into, and — where possible — ways to overcome them.


### Limitation #1: The admin cannot display nested inlines


Let's say you have three models called `Group`, `Person` and `EmailAddress`. A `Group` can have multiple `Person`s, and a `Person` can have multiple `EmailAddress`es. You might be tempted to do this in your `admin.py`:

    
    # In admin.py.
    
    class EmailAddressInline(admin.TabularInline):
        model = models.EmailAddress
        extra = 1
    
    class PersonInline(admin.TabularInline):
        model = models.Person
        extra = 3
        inlines = [
            EmailAddressInline,
        ]
    
    class GroupAdmin(admin.ModelAdmin):
        inlines = [
            PersonInline,
        ]
    
    admin.site.register(models.Group, GroupAdmin)


This will not work. The admin framework doesn't expect to ever find an `InlineModelAdmin` nested inside another `InlineModelAdmin`, so the `inlines` list defined on `PersonInline` makes no sense to it and it will happily ignore it. When you try to edit or add a `Group` in your admin, you will see a list fields for editing or adding related `Person`s at the bottom of the page, and that's it. The list of fields for editing or adding `EmailAddress`es that you expected to see attached to each `Person` will not appear in the admin UI.

If you want nested inlines, you will have to extend the admin template for your model to handle them. You will also have to create a custom `ModelForm` subclass to deal with the data that you get from your new template. See [this StackOverflow question](http://stackoverflow.com/questions/3681258/nested-inlines-in-the-django-admin) for some more details.

Support for displaying nested inlines in the admin is an oft-requested feature that even has a working patch, but it hasn't made its way into Django yet because it doesn't meet the community's quality standards. See [ticket #9025](https://code.djangoproject.com/ticket/9025).


### Limitation #2: All staff users can access all AdminSite instances


If you have multiple admin sites in your project, then all `User`s who have their `is_staff` flag set to `True` have access to all those admin sites. Often, this is not desirable. For example, if you're building a forum application, you might want to have separate dashboards for forum administrators and forum moderators. Or, if you're building an app for managing your company's payroll, you might want separate dashboards for accountants and employees. You don't want someone from one of these groups to accidentally stumble across the other group's admin interface.

The solution I use to get around this limitation is a little obtuse, but it works. First, I create a new model to serve as my `AUTH_PROFILE_MODULE`:

    
    # In models.py.
    
    class ForumUserProfile:
        pass
    # In settings.py.
    
    AUTH_PROFILE_MODULE = 'appname.ForumUserProfile'


On this model I define one permission for each admin site in my project:

    
    class ForumUserProfile:
        class Meta:
            permissions = (
                ('access_user_admin', 'Can access user admin.'),
                ('access_mod_admin', 'Can access moderator admin.'),
                ('access_admin_admin', 'Can access main admin.'),
            )


Then, I create a middleware that returns an HTTP 403 error if a user tries to access an admin he doesn't have the permission to access:

    
    # In middleware.py.
    
    class RestrictAdminMiddleware(object):
        RESTRICTED_URLS = (
            ('/user/', 'access_user_admin'),
            ('/mod/', 'access_mod_admin'),
            ('/admin/', 'access_admin_admin'),        
        )
    
        def process_request(self, request):
            user = request.user
    
            # If user is not authenticated, we'll let the admin
            # framework deal with him. We just care about staff users.
            if not user.authenticated: return None
    
            # Superusers are allowed to access everything, so we'll
            # let them through.
            if user.is_superuser(): return None
    
            if user.is_staff():
                for url in self.RESTRICTED_URLS:
                    if re.findall(url[0], request.path):
                        if not user.has_perm(url[1]):
                            raise PermissionDenied()
    
            return None
    # In settings.py.
    
    MIDDLEWARE_CLASSES = (
        'django.middleware.common.CommonMiddleware',
        'django.contrib.sessions.middleware.SessionMiddleware',
        'django.middleware.csrf.CsrfViewMiddleware',
        'django.contrib.auth.middleware.AuthenticationMiddleware',
        'django.contrib.messages.middleware.MessageMiddleware',
    
        # Our shiny new middleware!
        'appname.middleware.RestrictAdminMiddleware',
    )


Finally, I set up the appropriate permissions for my users in the admin and I'm good to go.


### Limitation #3: It is not possible to reorder, group or hide models on the admin index page without extending the index template


When the number of models you want to manage using the admin becomes large, it becomes necessary to organize them into logical groups for easy navigation. The admin currently groups models by app, which is fine when your apps have a small number of models. This organization scheme quickly becomes unwieldy when the number of models in a single app grows large. It becomes even worse when you have two related models in two separate apps that you think should logically be displayed under one group.

While the admin app itself doesn't currently have this functionality, it is easy to add. You can:



	
  1. Extend the admin index template (`admin/index.html`) and create groups yourself.

	
  2. Use a third party app that allows you to customize the admin using a straightforward Python API. Two such apps are [django-admin-tools](https://bitbucket.org/izi/django-admin-tools/) and [django-grappelli](https://github.com/sehmaschine/django-grappelli/).


I personally use [django-grappelli](https://github.com/sehmaschine/django-grappelli/) on all my websites.


### Limitation #4: ImageFields are not displayed as thumbnails


If you have an `ImageField` on any of your models, nine times out of ten you want to see a thumbnail of the image when you're editing an existing instance of that model. There are several third party apps that add this feature to the admin, and [the Django wiki lists a variety of solutions](https://code.djangoproject.com/wiki/ThumbNails).


### Limitation #5: Select boxes cannot be "chained"


This is a use-case that occurs very often. You have two select boxes on a page, and the list of choices in the second select box depends on the choice the user has made in the first select box. For example, the first select box could contain a list of Indian states and the second one a list of cities. You want to dynamically change the list of cities in the second box depending on which state is selected in the first box. The Django admin does not allow you to do that out of the box.

Fortunately, the solution is very simple: use [django-smart-selects](https://github.com/digi604/django-smart-selects).


### Limitation #6: "Save and continue editing" reloads the page


I think the purpose of the "Save and continue editing" button is lost if it reloads the page every time you click it. If you're editing a complex model that has many fields, reloading the page means you lose your position on the page and have to scroll around to find the field you were editing. While composing long-form text in a <textarea> — like I'm doing right now — clicking "Save and continue editing" means losing your position within the five-hundred or so words you just wrote. This is not an issue if you use the admin for one-off edits, but it becomes a major usability problem if you spend most of your time in the admin.

A full solution involves quite a bit of JavaScript, but you can get halfway there with a simple asynchronous POST request. Here's an example:

    
    # In admin.py.
    
    class JournalAdmin(admin.ModelAdmin):
        class Media:
            js = ('ajax_submit.js',)
    # In ajax_submit.js.
    
    # NOTE: I'm not very good at writing idiomatic JavaScript.
    # If you think this snippet can be improved, do
    # let me know.
    
    # WARNING: this code is meant for demonstration purposes. 
    # Do not use it in production. It fails on several edge cases
    # and, in the process, destroys your data, empties your bank
    # account, kidnaps your children and takes down reddit for a
    # month.
    
    "use strict;";
    
    var AJAXSubmit = function () {
        if (!$) {
            var $ = django.jQuery;
        }
    
        function ajax_submit(e) {
            e.preventDefault();
    
            var data = {
                // Don't forget the CSRF middleware token!
                "csrfmiddlewaretoken":
                    $("input[name='csrfmiddlewaretoken']").val(),
                "title": $("textarea#id_title").val(),
                "slug": $("input#id_slug").val(),
                "author": $("select#id_author").val(),
                "published_on_0": $("input#id_published_on_0").val(),
                "published_on_1": $("input#id_published_on_1").val(),
                "content": $("textarea#id_content").val()
            };
    
            if ($("input#id_published").is(":checked")) {
                data["id_published"] = "on";
            }
    
            $.ajax({
                type: "POST",
                url: "",
                data: data,
                success: function() {
                    alert("Saved!");
                }
            });
    
            return false;
        }
    
        $(document).ready(function() {
            var btn = $("div.submit-row input[name='_continue']");
            btn.click(ajax_submit);
        });
    }();


**Warning:** don't use this code in production. I've only tested it on Firefox and Chrome, where it fails on several edge-cases. If you use this in production and lose data, don't blame me.


### Limitation #7: Generic relationships are displayed poorly


Currently, the admin treats generic relationships just like any other data. Consider this:

    
    # In models.py.
    
    class TaggedItem(models.Model):
        tag = models.SlugField()
        content_type = models.ForeignKey(ContentType)
        object_id = models.PositiveIntegerField()
        content_object = generic.GenericForeignKey()


In the admin for `TaggedItem`, `content_type` will be displayed like any other foreign key: a select box containing a list of all content types in your project. `object_id` will not get any special treatment either: it will be displayed as a simple <input> box. This kind of treatment makes it impossible to figure out at a glance which object the generic foreign key actually refers to.

What I'd like the admin to do here is display the URL returned by the `content_object`'s `get_absolute_url()` in a read-only field right after the `object_id` field. Alternatively, the read-only URL field could simply contain a link to the `content_object`'s admin page.


### Limitation #8: While editing a model instance with a relationship to another model, it is possible to add a new instance of the other model to participate in the relationship, but impossible to edit an existing related instance


Let's say you've got a `BlogPost` model with a `ManyToMany` relationship to a `Tag` model. While editing your `BlogPost` in the admin, you see a nice multi-select box where you can choose the `Tag`s you want to associate with your `BlogPost`. There's a small "+" button next to the multi-select box. Clicking this button opens a pop-up window where you can add new `Tag` instances. Now, what happens when you create a new tag using this feature but accidentally give it an incorrect name? You'll find that there's no edit button next to the multi-select box. To fix the name of your newly-created tag, you will have to navigate to the admin page for that tag and fix the name from there. This is another minor inconvenience that can become a major usability issue if you use admin a lot.

I haven't sorted this issue out yet, but I can think of a couple of solutions that could work. I'll post them here in a separate blog post after I've implemented them in my own apps.


### Closing Remarks


The Django admin app is incredibly useful. With a little tweaking, it makes writing custom management interfaces for your webapps completely unnecessary. Yes, it has some shortcomings, but most of them can be overcome with a little extra code. Besides, Django development happens at a very fast pace, so you can expect some of these shortcomings to be fixed in future releases of the framework.


