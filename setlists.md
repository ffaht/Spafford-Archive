---
layout: default
title: All Setlists
---

# All Setlists

{% for setlist in site.setlists %}
<div class="setlist-preview">
    <h3><a href="{{ setlist.url }}">{{ setlist.title }}</a></h3>
    <div class="setlist-meta">
        <span class="date">{{ setlist.date | date: "%B %d, %Y" }}</span>
        {% if setlist.venue %} • <span class="venue">{{ setlist.venue }}</span>{% endif %}
        {% if setlist.city %} • <span class="location">{{ setlist.city }}{% if setlist.state %}, {{ setlist.state }}{% endif %}</span>{% endif %}
    </div>
</div>
{% endfor %}

<style>
.setlist-preview {
    margin-bottom: 20px;
    padding: 15px;
    border: 1px solid #eee;
    border-radius: 5px;
}

.setlist-preview h3 {
    margin: 0 0 5px 0;
}

.setlist-meta {
    color: #666;
    font-size: 0.9em;
}
</style>