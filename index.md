---
layout: page
home: true
---
<p id="mission">
  We develop <strong>scalable and self-improving AI systems and methodologies towards the goal of AGI</strong>, leveraging techniques in machine learning, systems, natural language processing, and beyond.
</p>

<p id="ad">
    <h2><strong><b>Prospective Students and Postdocs:</b></strong></h2>
    <ul>
      <li>For Stanford students interested in working in the lab, please check out <a href="/openings">project openings</a> and reach out to the project lead.</li>    
      <!-- <li>For students interested in working in the lab, please fill out <a href="https://docs.google.com/forms/d/e/1FAIpQLSemsYIAG1wUpGgclG1eaQxDNYxPHB5VZzLl9eRRFsT6NPa_0A/viewform">the following form</a>.</li> -->
      <li>If you are a prospective PhD student: please indicate your interest in working with Prof. Mirhoseini in the Stanford CS PhD online application form, as well as in your research statement. We admit new PhD students each year. Please do not contact us directly regarding PhD applications until admission decisions are out, as we will not be able to reply to emails from individual applicants.</li>
      <li>If you are a postdoc candidate, please reach out directly to Prof. Mirhoseini.</li>
    </ul>
  </p>

<div id="home" class="pure-g">
  <div id="themes" class="pure-u-1 pure-u-md-3-5">
    <h2>Research Themes</h2>
    {% for theme in site.data.research_themes %}
      <div id="theme-{{theme.key}}" class="theme" data-url="{{theme.url}}" data-people="{{theme.people}}" style="cursor: default;">
        <!-- <img src="/themes/{{theme.key}}.png" style="max-width: 100%; height: auto; display: block; margin-top: 0;"> -->
          <div style="padding-top: 0px; border-radius: 5px; margin-bottom: 0px;">
            <div class="content">
              <!-- disable link highlight as we don't have page to point to yet -->
              <h3 style="pointer-events: none;">{{theme.name}}</h3>
              {{theme.desc | markdownify}}
            </div>
          </div>
      </div>
    {% endfor %}

    <h2 style="margin-top: 2em;">Courses</h2>
    {% for course in site.data.courses %}
      <div id="course-{{course.key}}" class="theme" style="cursor: pointer;" onclick="window.open('{{course.url}}', '_blank')">
          <div style="padding-top: 0px; border-radius: 5px; margin-bottom: 0px;">
            <div class="content">
              <h3>{{course.name}}</h3>
              {{course.desc | markdownify}}
            </div>
          </div>
      </div>
    {% endfor %}
  </div>

  <div class="pure-u-1 pure-u-md-2-5">

    <div id="pinned">
      {% assign pinned = site.data.news | where: "pinned", true %}
      {{pinned[0].desc | markdownify}}
    </div>

    <h2 id="people-header">People</h2>
    <div id="people" class="pure-g">
      {% assign members = site.data.people | filter_alumni: nil | sort_people: 'Professor, PhD, Visiting, Researcher', true %}
      {% for person in members %}
        {% unless person[1].not_current %}
          {% unless person[1].title contains 'Master' or person[1].title contains 'Undergraduate' %}
            {% unless person[1].on_leave %}
              <div id="{{person[0]}}" class="person pure-u-1-4">
                <a href="{{person[1].url}}">
                  <p class="headshot"><img src="/imgs/people/{{person[0]}}.jpg" alt="" /></p>
                  <p class="name">{{person[1].name}}</p>
                  <p class="title">{{person[1].title}}</p>
                  {% if person[1].note %}
                    <p class="note" style="font-size: 0.85em; color: #666; margin-top: 2px;">{{person[1].note}}</p>
                  {% endif %}
                </a>
              </div>
            {% endunless %}
          {% endunless %}
        {% endunless %}
      {% endfor %}
      <div class="pure-u-1" style="height: 0;"></div>
      {% for person in members %}
        {% unless person[1].not_current %}
          {% unless person[1].title contains 'Master' or person[1].title contains 'Undergraduate' %}
            {% if person[1].on_leave %}
              <div id="{{person[0]}}" class="person pure-u-1-4">
                <a href="{{person[1].url}}">
                  <p class="headshot"><img src="/imgs/people/{{person[0]}}.jpg" alt="" /></p>
                  <p class="name">{{person[1].name}}</p>
                  <p class="title">{{person[1].title}}</p>
                  {% if person[1].note %}
                    <p class="note" style="font-size: 0.85em; color: #666; margin-top: 2px;">{{person[1].note}}</p>
                  {% endif %}
                </a>
              </div>
            {% endif %}
          {% endunless %}
        {% endunless %}
      {% endfor %}
    </div>

    <h3 id="students-header">Masters & Undergraduate Researchers</h3>
    <div id="students" class="pure-g">
      {% assign students = site.data.people | filter_alumni: nil %}
      {% for person in students %}
        {% unless person[1].not_current %}
          {% if person[1].title contains 'Master' or person[1].title contains 'Undergraduate' %}
            <div class="person-name pure-u-1-3" style="margin-bottom: 10px;">
              <a href="{{person[1].url}}">{{person[1].name}}</a>
            </div>
          {% endif %}
        {% endunless %}
      {% endfor %}
    </div>
    <h3 id="alumni-header">Alumni</h3>
    <ul id="alumni" class="pure-g" style="list-style: none; padding: 0; text-align: left;">
      {% assign alumni = site.data.people | filter_alumni: true | sort_people: 'PhD, Postdoctoral, Scientist' %}
      {% for person in alumni  %}
        <li id="{{person[0]}}" class="person pure-u-1-2" style="margin-bottom: 8px;">
          <a href="{{person[1].url}}" style="border-bottom: none; display: inline;">{{person[1].name}}</a> ({{person[1].title}}{% if person[1].next %}, Next: {{person[1].next}}{% endif %})
        </li>
      {% endfor %}
    </ul>
  </div>
</div>

<div>
  <h5>Design from <a href="https://vis.csail.mit.edu/">MIT Visualization Group</a></h5>
</div>
