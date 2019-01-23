---
layout: default
---

# {{ site.title }}

## Spring 2019

### Meets Tuesday, Thursday 1:15 - 2:35 AM in ES 223

### Instructor: Brian Rose
- **Email**: <brose@albany.edu>
- **Office**: ES 315
- **Office hours**: drop-in or by appointment
- **Website**: <http://www.atmos.albany.edu/facstaff/brose>

### Course website:
<http://www.atmos.albany.edu/facstaff/brose/classes/ATM623_Spring2019/>

Look below to find news and updates about the course. Check back here often.



<div class="home">

  <ul class="post-list">
    {% for post in site.posts %}
      <li>
        <!-- <span class="post-meta">{{ post.date | date: "%b %-d, %Y" }}</span> -->
        <a class="post-link" href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>
      </li>
    {% endfor %}
  </ul>
</div>
