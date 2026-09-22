---
layout: list
title: Bugku Writeups
---
# Bugku
Web、Misc、Crypto、Reverse、PWN等入门题目的解题记录。

{% assign bugku = site.posts | where_exp:"item","item.categories[0] == 'Bugku'" %}

## Web
{% assign web = bugku | where_exp:"item","item.categories[1] == 'Web'" %}
{% for post in web %}
- [{{ post.title }}]({{ post.url }})
{% endfor %}

## Crypto
{% assign crypto = bugku | where_exp:"item","item.categories[1] == 'Crypto'" %}
{% for post in crypto %}
- [{{ post.title }}]({{ post.url }})
{% endfor %}

## Misc
{% assign misc = bugku | where_exp:"item","item.categories[1] == 'Misc'" %}
{% for post in misc %}
- [{{ post.title }}]({{ post.url }})
{% endfor %}

## Reverse
{% assign reverse = bugku | where_exp:"item","item.categories[1] == 'Reverse'" %}
{% for post in reverse %}
- [{{ post.title }}]({{ post.url }})
{% endfor %}

## PWN
{% assign pwn = bugku | where_exp:"item","item.categories[1] == 'PWN'" %}
{% for post in pwn %}
- [{{ post.title }}]({{ post.url }})
{% endfor %}
