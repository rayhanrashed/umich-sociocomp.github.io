---
title: Publications
layout: page
---
<div id="pubs" class="pure-g">
  <div id="content" class="pure-u-1 pure-u-md-3-4">
    <h1 class="title">Publications</h1>

    {% assign pubYears = site.pubs | group_by:"year" | sort: "date" | reverse %}
    {% for year in pubYears %}
      <div id="year-{{year.name}}" class="year pure-g">
        <div class="pure-u-1-3 pure-u-md-1-5"></div>
        <div class="pure-u-2-3 pure-u-md-4-5">
          <h2>{{year.name}}</h2>
        </div>
      </div>

      {% assign pubs = year.items | sort: 'date' | reverse %}
      {% for pub in pubs %}
        {% if pub.redirect %}
          {% assign url = pub.redirect %}
        {% elsif pub.external_url %}
          {% assign url = pub.external_url %}
        {% else %}
          {% assign url = pub.url | relative_url | replace: 'index.html', '' %}
        {% endif %}
        <div id="{{pub.slug}}" class="pub pure-g" data-pub='{"id":"{{pub.slug}}","title":"{{pub.title | escape}}","year":{{pub.year}},"type":"{{pub.type}}","tags":[{% for tag in pub.tags %}"{{tag | downcase}}"{% unless forloop.last %},{% endunless %}{% endfor %}],"authors":[{% for author in pub.authors %}{% assign person = site.data.people[author.key] %}"{{author.name | default: person.name | escape}}"{% unless forloop.last %},{% endunless %}{% endfor %}]}'>
          <div class="thumbnail pure-u-1-3 pure-u-md-1-5">
            <a href="{{url}}">
              <img src="{{site.baseurl}}/imgs/thumbs/{{pub.slug}}.png" alt="" />
            </a>
          </div>

          <div class="pure-u-2-3 pure-u-md-4-5">
            <h3><a href="{{url}}">{{pub.title}}</a></h3>
            <p class="authors">
            {% for author in pub.authors %}
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
              {% if pub.preprint %}
                {{pub.preprint.server}}: {{pub.preprint.id}}
              {% else %}
                {% if pub.venue != "none" %}
                {{site.data.venues[pub.venue].full}}, {{pub.year}}
                {% else %}
                {{pub.year}}
                {% endif %}
              {% endif %}
            </p>
            {% if pub.award %}
              <p class="award">
                <i class="fas fa-award"></i> {{pub.award}}
              </p>
            {% endif %}
            <p class="links">
              {% for material in pub.materials %}
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

    <h4>Publication Type</h4>
    <div id="types">
      {% assign types = site.pubs | map: 'type' | uniq %}
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

<script>
  {% include pubsearch.js %}
</script>
