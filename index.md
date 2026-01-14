---
layout: post
title: Home
---

<div class="flag-filters">
  <span>Filter by flags:</span>
  <a href="#" data-flag="miis" class="flag-filter">--miis</a>
  <a href="#" data-flag="decompilation" class="flag-filter">--decompilation</a>
  <a href="#" data-flag="commentary" class="flag-filter">--commentary</a>
  <a href="#" data-flag="unhinged" class="flag-filter">--unhinged</a>
</div>

<script>
  // Parse URL parameters to get active flags and author
  const urlParams = new URLSearchParams(window.location.search);
  let activeFlags = urlParams.get('flags') ? urlParams.get('flags').split(',') : [];
  const filterAuthor = urlParams.get('author');
  
  // Apply active state on page load
  document.addEventListener('DOMContentLoaded', function() {
    // Highlight active filters
    document.querySelectorAll('.flag-filter').forEach(function(link) {
      const flag = link.getAttribute('data-flag');
      if (activeFlags.includes(flag)) {
        link.classList.add('active');
      }
    });
    
    // Filter posts
    filterPosts();
    
    // Add click handlers
    document.querySelectorAll('.flag-filter').forEach(function(link) {
      link.addEventListener('click', function(e) {
        e.preventDefault();
        const flag = this.getAttribute('data-flag');
        
        // Toggle flag
        const index = activeFlags.indexOf(flag);
        if (index > -1) {
          activeFlags.splice(index, 1);
          this.classList.remove('active');
        } else {
          activeFlags.push(flag);
          this.classList.add('active');
        }
        
        // Update URL
        if (activeFlags.length > 0) {
          const newUrl = '?flags=' + activeFlags.join(',');
          window.history.pushState({}, '', newUrl);
        } else {
          window.history.pushState({}, '', window.location.pathname);
        }
        
        // Filter posts
        filterPosts();
      });
    });
  });
  
  function filterPosts() {
    document.querySelectorAll('.post-preview').forEach(function(post) {
      let showPost = true;
      
      // Filter by flags
      if (activeFlags.length > 0) {
        const postFlags = post.getAttribute('data-flags');
        let hasMatch = false;
        
        if (postFlags) {
          activeFlags.forEach(function(flag) {
            if (postFlags.includes(flag)) {
              hasMatch = true;
            }
          });
        }
        
        showPost = hasMatch;
      }
      
      // Filter by author
      if (filterAuthor && showPost) {
        const postAuthors = post.getAttribute('data-authors');
        if (postAuthors) {
          showPost = postAuthors.toLowerCase().includes(filterAuthor.toLowerCase());
        } else {
          showPost = false;
        }
      }
      
      post.style.display = showPost ? 'block' : 'none';
    });
  }
</script>

<br>

# Recent Posts

{% for post in site.posts %}
  {% unless post.other_category %}
  <article class="post-preview" data-flags="{{ post.flags | join: ' ' }}" data-authors="{% if post.authors %}{{ post.authors | join: ', ' }}{% elsif post.author %}{{ post.author }}{% endif %}">
    <h2><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
    <p class="post-date"><small>{{ post.date | date: "%B %d, %Y" }}</small></p>
    {% if post.flags %}
    <div class="post-flags">
      {% for flag in post.flags %}
      <span class="flag-badge">--{{ flag }}</span>
      {% endfor %}
    </div>
    {% endif %}
    {% if post.thumbnail %}
      <a href="{{ post.url | relative_url }}">
        <img src="{{ post.thumbnail | relative_url }}" alt="{{ post.title }}" class="post-thumbnail">
      </a>
    {% endif %}
  </article>
  {% endunless %}
{% endfor %}

<br>
<br>
# Other

{% for post in site.posts %}
  {% if post.other_category %}
  <article class="post-preview" data-flags="{{ post.flags | join: ' ' }}" data-authors="{% if post.authors %}{{ post.authors | join: ', ' }}{% elsif post.author %}{{ post.author }}{% endif %}">
    <h2><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
    <p class="post-date"><small>{{ post.date | date: "%B %d, %Y" }}</small></p>
    {% if post.flags %}
    <div class="post-flags">
      {% for flag in post.flags %}
      <span class="flag-badge">--{{ flag }}</span>
      {% endfor %}
    </div>
    {% endif %}
    {% if post.thumbnail %}
      <a href="{{ post.url | relative_url }}">
        <img src="{{ post.thumbnail | relative_url }}" alt="{{ post.title }}" class="post-thumbnail">
      </a>
    {% endif %}
  </article>
  {% endif %}
{% endfor %}
