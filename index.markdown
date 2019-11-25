---
layout: home
---

I [build software](/projects) and [play music](//rawfunkmaharishi.uk), and I have a [non-working pancreas](//www.diabetes.org.uk/Guide-to-diabetes/What-is-diabetes/What-is-Type-1-diabetes/).

<dl>
{% for item in site.data.navigation %}
  <dt><a href="{{ item.path }}">{{ item.title }}</a></dt>
  <dd>{{ item.description }}</dd>
{% endfor %}
</dl>