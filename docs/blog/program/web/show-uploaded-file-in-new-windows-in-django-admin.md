---
date: '2017-04-13 10:21:06'
draft: false
lastmod: '2017-11-25 20:16:11'
tags:
- program
- web
title: How to make the new uploaded file in Django admin UI to open in a new window
---

By default, when use Django admin to upload a file/image in inline models, the url it got is:

===

	<p class="file-upload">
	Currently: <a href="/upload/ct_attachment/2014/09/18/cloud_SL4WKba.jpg">ct_attachment/2014/09/18/cloud_SL4WKba.jpg</a>
	Change:    <input id="id_attachments-0-files" type="file" name="attachments-0-files"/>
	</p>


When one click the URL to download the file already uploaded, it will open in the same window, which means the admin UI will be replaced with the file downloading window.
Sometimes, we just need to check what's the file it is, so open it in a new windows is much better.
Add ```target=_blank``` in ```<a href=....``` is enough, but how to add it?

Forunately, there is a ```formfield_overrides``` variable can be overrided in the AdminModel.
By checking django source codes, if we define a customized widget, override the url_markup_template variable which defined in django.forms.ClearableFileInput, then we can do a trick to implement this change, e,g:

    class FileInNewWindowWidget(admin.widgets.AdminFileWidget):
        # AdminFileWidget inherits from django.forms.ClearableFileInput
        # The original url_markup_template in django.forms.ClearableFileInput is:
        # url_markup_template = '<a href="{0}">{1}</a>'
        url_markup_template = '<a href="{0}" target="_blank">{1}</a>'

    class AttachmentInline(admin.TabularInline):
        formfield_overrides = {
            models.FileField: {'widget': FileInNewWindowWidget},
        }

    class CustomerAdmin(admin.ModelAdmin):
        inlines = [AttachmentInline]