---
layout: default
title: Home
nav_order: 1
description: "Suresh Bojjam's Technical Blog - Java Development, Programming Tutorials, and Software Engineering Insights"
permalink: /
---

# Welcome to Suresh Bojjam's Technical Blog
{: .fs-9 }

Exploring Java Development, Programming Best Practices, and Software Engineering
{: .fs-6 .fw-300 }

[Get started now](#getting-started){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
[View on GitHub](https://github.com/suresh-bojjam/suresh-bojjam){: .btn .fs-5 .mb-4 .mb-md-0 }

---

## Getting Started

This blog focuses on featuring comprehensive tutorials and insights into:

### 🚀 Featured Topics

- **Java 8+/17/21 Features**: Lambda expressions, streams, method references, and functional programming & etc.
- **Spring Way Of Coding**:   Springboot developement.
- **Building Agents**: Agents the pairprogrammer in building your software and products.

### 📚 Recent Posts

{% for post in site.posts limit:3 %}
- [{{ post.title }}]({{ post.url | relative_url }}) - {{ post.date | date: "%B %d, %Y" }}
{% endfor %}

[View all posts →](/posts){: .btn .btn-outline }

---

## About This Site

This technical blog is built with:

- **Jekyll** - Static site generator
- **Just the Docs** - Clean, responsive documentation theme
- **GitHub Pages** - Free hosting and deployment
- **Markdown** - Simple content creation

### Navigation

{: .note }
Use the navigation menu on the left to explore different topics. All content is organized by categories for easy browsing.

### Search

{: .highlight }
Use the search functionality (Ctrl+K or Cmd+K) to quickly find specific topics or code examples.

---

## Contact & Connect

- **Email**: [bojjamsuresh@gmail.com](mailto:bojjamsuresh@gmail.com)
- **GitHub**: [View my repositories](https://github.com/suresh-bojjam)
- **LinkedIn**: Connect with me for professional discussions

---

{: .fs-3 }
*Happy coding! 🎯*
