---
date: '2017-04-13 10:21:06'
draft: false
lastmod: '2017-11-25 20:16:11'
tags:
- program
- web
title: Why some translation not shown up in Django app
---

While I dev some django app, I follow the standard procedure to generate the message.po file

    python djang-admin.py makemessages -l zh_CN

I tried compile it and restart httpd to clear the django translation cache, but some text still not shown up correctly with the translated text, at the first galance, I think this may be some translation issue, tried many ways to disable gettext/django translation cache, still no luck. Then I checked the comiled .mo files, it seems doesn't contain the translated key,

	strings /path/to/trasnlated.mo | grep <msgid></msgid>

I found that all missed translation text all have ", Fuzzy' attribute in .po file,
by google searching, the 'Fuzzy' attribute indicates the text translation will be skipped, oops, that's the reason.

- What “fuzzy” means – python, django, gettext.
	
	!!! A django application is in process of adding support for second language.
    !!! During adding translations to the django.po file after next .po file compilation some of previously translated texts were displayed on target site in their original, untranslated form.
	!!! "#, fuzzy" flag was responsible for not translating my strings.
	!!! Fuzzy flag description. In short words: if you are sure that “fuzzy” marked translation is correct simply remove "fuzzy" flag, compile the .po file and you are done.