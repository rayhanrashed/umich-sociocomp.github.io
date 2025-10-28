---
title: Blogs
layout: page
---
<div id="blogs" class="pure-g">
  <div id="content" class="pure-u-1 pure-u-md-3-4">
    <h1 class="title">Blogs</h1>

    {% assign blogYears = site.blogs | group_by:"year" | sort: "date" | reverse %}
    {% for year in blogYears %}
      <div id="year-{{year.name}}" class="year pure-g">
        <div class="pure-u-1-3 pure-u-md-1-5"></div>
        <div class="pure-u-2-3 pure-u-md-4-5">
          <h2>{{year.name}}</h2>
        </div>
      </div>

      {% assign blogs = year.items | sort: 'date' | reverse %}
      {% for blog in blogs %}
        {% if blog.redirect %}
          {% assign url = blog.redirect %}
        {% elsif blog.external_url %}
          {% assign url = blog.external_url %}
        {% else %}
          {% assign url = blog.url | relative_url | replace: 'index.html', '' %}
        {% endif %}
        <div id="{{blog.slug}}" class="blog pure-g" data-blog='{"id":"{{blog.slug}}","title":"{{blog.title | escape}}","year":{{blog.year}},"type":"{{blog.type}}","tags":[{% for tag in blog.tags %}"{{tag | downcase}}"{% unless forloop.last %},{% endunless %}{% endfor %}],"authors":[{% for author in blog.authors %}{% assign person = site.data.people[author.key] %}"{{author.name | default: person.name | escape}}"{% unless forloop.last %},{% endunless %}{% endfor %}]}'>
          <div class="thumbnail pure-u-1-3 pure-u-md-1-5">
            <a href="{{url}}">
              <img src="{{site.baseurl}}/imgs/thumbs/{{blog.slug}}.png" alt="" />
            </a>
          </div>

          <div class="pure-u-2-3 pure-u-md-4-5">
            <h3><a href="{{url}}">{{blog.title}}</a></h3>
            <p class="authors">
            {% for author in blog.authors %}
              {% assign person = site.data.people[author.key] %}
              {% assign name = author.name | default:person.name %}
              {% if person.url %}
                <a href="{{person.url}}">{{name}}</a>{% if author.equal %}*{% endif %}{% unless forloop.last %}, {% endunless %}
              {% else %}
                {{name}}{% if author.equal %}*{% endif %}{% unless forloop.last %}, {% endunless %}
              {% endif %}
            {% endfor %}
            </p>

            <p class="venue">
              {% if blog.preprint %}
                {{blog.preprint.server}}: {{blog.preprint.id}}
              {% else %}
                {% if blog.venue != "none" %}
                {{site.data.venues[blog.venue].full}}, {{blog.year}}
                {% else %}
                {{blog.year}}
                {% endif %}
              {% endif %}
            </p>
            {% if blog.award %}
              <p class="award">
                <i class="fas fa-award"></i> {{blog.award}}
              </p>
            {% endif %}
            <p class="links">
              {% for material in blog.materials %}
                {% if forloop.first %}
                  <a href="{{material.url}}">
                {% else %}
                  &middot; <a href="{{material.url}}">
                {% endif %}
                   {{material.name}}
                </a>
              {% endfor %}
            </p>
          </div>
        </div>
      {% endfor %}
    {% endfor %}
  </div> 
  
  <div id="sidebar" class="pure-u-1 pure-u-md-1-4">
    <h4>Search</h4>
    <input type="text" id="search" placeholder="Search title or authors...">

    <h4>Blog Type</h4>
    <div id="types">
      {% assign types = site.blogs | map: 'type' | uniq %}
      {% for type in types %}
        <a id="type-{{type}}" class="tag" data-tag="{{type}}">{{type | capitalize}} <span>()</span></a>
      {% endfor %}
    </div>

    <h4>Tags</h4>
    <div id="tags">
      {% for group in site.data.tags %}
        <p>
          {% for tag in group %}
            <a id="tag-{{tag | replace: ' ', '-'}}" class="tag" data-tag="{{tag}}">
              {% assign words = tag | split: ' ' %}
              {% for word in words %}
                {% if word == 'human-ai' %}
                  Human-AI
                {% else %}
                  {{word | capitalize}}
                {% endif %}
              {% endfor %}
              <span>()</span>
            </a>
          {% endfor %}
        </p>
      {% endfor %}
    </div>
  </div>  
</div>
