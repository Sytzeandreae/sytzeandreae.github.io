---
layout: post
title: "Creating responsive content types for FeinCMS"
published: true
categories: [python, django]
---

A while ago, I needed to create a website on the Django platform in combination with [FeinCMS](http://www.feincms.org). For those who don't know, FeinCMS is a Django app, which allows you to create pages on a website relativly fast. It's an extremely stupid content management system, which knows nothing of content, just enough to create an admin interface and pages.

The way FeinCMS works is you create content types. Content types are abstract models, representing for instance a paragraph, a header or an image. In the admin interface, these are added as blocks to regions on pages. You can place as many of these blocks you'd like, which makes it great for creating content.

The only problem I had with this, is that these content types are not responsive by nature, and I would like to give these blocks a certain width on a certain screen size. So what I had to was find a way to make these responsive.

## The solution
I created a Django app named responsive\_content\_type, which will allow me to create and render responsive interfaces.

The models.py contains the base model for our content type. It adds a couple of fields, holding the information for the number of column on large, medium and small screensizes. The render function in this file looks for the template of the classname. This way, the models we extend from this don't need this function.

``` python
# models.py
from django.db import models
from django.utils.translation import ugettext_lazy as _
from django.template.loader import render_to_string


class ResponsiveContentType(models.Model):
    small = models.IntegerField(_('Small'), max_length=2, 
                                choices=[(i, 'small-%d' % i) for i in xrange(1, 13)],
                                help_text=_('Number of columns on a small screen.'))
    medium = models.IntegerField(_('Medium'), max_length=2, 
                                choices=[(i, 'medium-%d' % i) for i in xrange(1, 13)],
                                help_text=_('Number of columns on a medium screen.'))
    large = models.IntegerField(_('Large'), max_length=2, 
                                choices=[(i, 'large-%d' % i) for i in xrange(1, 13)],
                                help_text=_('Number of columns on a large screen.'))

    def render(self, **kwargs):
        return render_to_string(
        	'content/%s.html' % self.__class__.__name__.lower().split('_')[-1],
        	{
                'content': self,
                'request': kwargs.get('request'),
            }
        )

    class Meta:
        abstract = True
```

FeinCMS normally offers a template tag called render\_region, rendering all the content added to a region on a certain page. I created a new template tag which will take this functionality and render the region in a responsive way. Since I use foundation for a lot of my websites, the main number of columns a row can handle is 12. This template tag will calculate the number of columns and put the content in rows containing maximum 12 columns.

``` python
# templatetags/responsive_tags.py
...
from feincms.templatetags.feincms_tags import _render_content
...
def _responsive_render_content(content, **kwargs):
    r = _render_content(content, **kwargs)
    return render_to_string('responsive/wrapper.html', {
        'r': r,
        'large': content.large,
        'medium': content.medium,
        'small': content.small
    })
...
@register.simple_tag(takes_context=True)
def render_responsive_region(context, feincms_object, region, request=None):
    i = [i.large if i.large else i.medium if i.medium else i.small for i in getattr(feincms_object.content, region)]
    n = []
    u = 1
    while len(i) > 0:
        if sum(i[:u]) > 12:
            n.append(i[:u-1]) if len(i[:u]) > 1 else i[:u]
            i = i[u-1:] if len(i[:u]) > 1 else i[:u]
            u = 0
        elif sum(i) <= 12: # If we have less than 12 columns left, append it all to one row
            n.append(i)
            i = []
        u += 1
    
    content_items = getattr(feincms_object.content, region)
    occur = 0
    content = []
    for container in n:
        r = ''
        for l in container:
            r += _responsive_render_content(content_items[occur], request=request, context=context)
            occur += 1
        content.append(r)

    return render_to_string('responsive/row.html', {'content': content})
```

In these functions there are two html files used: wrapper.html and row.html. Wrapper.html will wrap our content type in a block, containing the number of columns we have set in our model.

``` html
<div class="small-12 medium-8">
    <p>Lorem ipsum...</p>
</div>
```

The row.html containes items up to 12 columns. It uses the biggest screen size to check this. So it will look as follows:

``` html
<div class="row">
    <div class="small-12 medium-8 large-8">
        <p>Lorem ipsum...</p>
    </div>
    <div class="small-12 medium-6 large-4">
    	<img src="image.jpg" alt="Lorem ipsum..." />
    </div>
</div>
````

This is the gist of the way I created responsive content types in FeinCMS. I'm working on making this a package in pip. Feel free to check out the [github repo](https://github.com/sytzeandreae/FeinCMS-Responsive-Content-Type)!