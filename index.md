---
layout: page
title: The Elegant Network
tagline: Perspectives on Networking Infrastructure Design and Practice
---
This is a series of blog posts on the [authors'](/pages/authors.html) perspectives on network infrastructure design and practice. The authors experience is in data center networks, and the primary focus will therefore be the data center. This blog also serves as a companion to a book written by one of the authors, [Cloud Native Data Center Networking](https://www.amazon.com/Cloud-Native-Data-Center-Networking/dp/1492045608/). The posts cover everything from musings to code on various aspects of network infrastructure, from network design to routers to Kubernetes networking to open source tools such as routing suites and observability and troubleshooting tools. You can also follow us on Medium at [The Elegant Network](https://medium.com/the-elegant-network) publication.

## Posts

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
      <p>
      {{ post.excerpt }}
      </p>
    </li>
  {% endfor %}
</ul>
