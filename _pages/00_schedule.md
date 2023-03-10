---
layout: page
permalink: /schedule
title: Schedule
---

<!-- posts and pages used as sources -->
{% assign all = null | compact %}
{% assign all = all | concat:site.posts | concat:site.pages %}

<!-- Setup order for Units -->
{% assign units = "7,1,2,3,4,5,6,8,9" | split: ',' %}
{% for unit in units %}

  <!-- Each Unit has a range of weeks and a heading -->
  {% if unit == "1" %} 
      {% assign start = 0 %}
      {% assign end = 3 %}
## Unit {{unit}}: Introduction to Tools and Resources
  > To learn Java and build skills for Career Technical Education students will quickly immerse into Tools and Resources for Java Development and Fastpages Blogging.  These early weeks will focus on the Development Environment, Fastpages Blogging platform, Code.org resources, AP Classroom resources, and Programming Java with Jupyter Notebooks.

  {% elsif unit == "2" %} 
      {% assign start = 4 %}
      {% assign end = 7 %}
## Unit {{unit}}: Java Mini-labs
  > After using Code.org in the first unit, students have been introduced to Classes and Inheritance.  In this unit students will become more familiar with Java development through mini-labs.  These labs will focus on AP required aspects of Java, additionally they can be used as code to support the backend of a Desktop App or WebSite. This unit concludes with 4 person Project Plan, kicking off the end of trimester N@TM project.   Students will be able to write code that completes full stack process of Frontend talking to the Java backend.  This section will conclude with a "required" N@tM open house.

  {% elsif unit == "3" %} 
      {% assign start = 8 %}
      {% assign end = 12 %}
## Unit {{unit}}: Individual Project
  > This Units requirement is to to build individual development skills in Java.  By the end of this unit students will be aware of all the College Board Units and FRQ types.   Student will participate or have participated in presentations and live grading of peers work.  Fastpages Blogs and Jupyter Notebooks are required for all individuals.   By the end of this section you should have examples, study materials, and code that show a great deal of effort, understanding, and competency.
      
  {% elsif unit == "4" %} 
      {% assign start = 13 %}
      {% assign end = 16 %}
## Unit {{unit}}: College Board Study Unit
> This period will complete formal teaching and grading on the basics of the 10 units.  Also, there will be a tech talk and homework on each FRQ type customized for PBL idea.

  {% elsif unit == "5" %} 
      {% assign start = 17 %}
      {% assign end = 20 %}
## Unit {{unit}}: 2nd Trimester Projects
> Objective of these weeks is to explore and create ideas and concepts for a Team Trimester N@tM project.  Adding frontend and creativity while using APIs/Databases.  Students should earn trust in these design weeks to instill confidence in the Teach for the right to work independently on a project of their own personal interest.

{% elsif unit == "6" %} 
      {% assign start = 21 %}
      {% assign end = 24 %}
## Unit {{unit}}: 2nd Trimester N@tM and finish Project
> This will be most creative portion of year for CSA students.  Each person within "Student Teams" will have their own specialty within their student project that shows Full Stack competency.  Intentions for this period is to have a free and creative period, driven by your Issues and Scrum Board.  Student should be able to talk about design, fe/be coding, and database features of their portion of the project in weekly live reviews.  

{% elsif unit == "7" %} 
      {% assign start = 25 %}
      {% assign end = 29 %}
## Unit {{unit}}: Trimester 3 Data Structures
> Trimester 3 has a focus Data Structures that relate to the AP exam.  A key requirement is to make your own Algo Rythmic sorting video.  The midterm project, due at the beginning of week 30, will be either FE/BE or Jupyter Notebook project and hopefully you can use it as a lesson starting Week 30.   The theme is produce work that can be used to help you pass the AP Exam and be used by future generations in AP CS, as a Study Aide.  Each student must cover key concepts from one of the Four AP FRQ types, contain a key Data Structure, and utilize sorting. These requirements are fairly generic and could complement any teaching assignment or project.

{% elsif unit == "8" %} 
      {% assign start = 30 %}
      {% assign end = 33 %}
## Unit {{unit}}: Trimester 3 Data Structures
> Trimester 3 AP unit. Test is May 3rd.  Student will lead several study sessions (20 minute test, follow by review) the week before the exam.  In any break in study, students will transition activities to a final project.

{% elsif unit == "9" %} 
      {% assign start = 34 %}
      {% assign end = 36 %}
## Unit {{unit}}: Trimester 3 Data Structures
> Trimester 3 Wrap up your preferred project or instructional site.  There will be an opportunity to contribute and be published to the NightHawk Coding Society library.  If your project is selected, then you will receive a high 'A' on final.

  {% endif %}

  <!-- Column Headings for Blogs -->
  <table>
      <tr>
        <th>Week</th>
        <th>Sprint/Points Link</th>
        <th>AP Test Prep</th>
        <th>Career Tech</th>
        <th>Human Prep</th>
      </tr>

  <!-- These loops group blogs according to Type and Week -->
  {% assign units = null | compact %}  <!-- empty array -->
  {% assign sym = "|||" %}  <!-- string/symbol used a separator  -->
  {% assign deli = sym | compact %} <!-- force to array element -->

  {% for i in (start..end) -%}
    {% assign pt = null | compact %} <!-- empty array -->
    {% assign ap = null | compact %}
    {% assign tt = null | compact %}
    {% assign hm = null | compact %}
    {% assign uk = null | compact %}

  <!-- looping through all posts -->
    {% for post in all %}

  <!-- prepare data blog post data for evaluation -->
      {% assign week = post.week | plus: 0 %}  <!-- force to integer -->
      {% assign title = post.title | compact %}
      {% assign url = post.url | compact %}

  <!-- process posts for current week -->
      {% if week == i %} 

  <!-- organizing blogs by type -->
        {% if post.type == "plan" %} 
            {% assign pt = pt | push: title %}
            {% assign pt = pt | push: url %}
        {% elsif post.type == "ap" %}
            {% assign ap = ap | push: title %}
            {% assign ap = ap | push: url %}  
        {% elsif post.type == "pbl" %}
            {% assign tt = tt | push: title %}
            {% assign tt = tt | push: url %} 
        {% elsif post.type == "human" %}
            {% assign hm = hm | push: title %}
            {% assign hm = hm | push: url %} 
        {% else %}
            {% assign uk = uk | push: title %}
            {% assign uk = uk | push: url %}     
        {% endif %}

      {% endif %}
    {% endfor %}

  <!-- ordering blogs and inserting column delimiters -->
  {% assign units = units | concat:pt | concat:deli | concat:ap | concat:deli | concat:tt | concat:deli | concat:hm %}

  <!-- Display documents by type-->
  <tr>
  <td> {{i}} </td> 
  <td>
  {% for i in (0..100) -%}   <!-- forever loop -->
    {% if units.size == 0 %} <!-- break loop when data is empty -->
      {% break %}
    {% elsif units[0] == sym %} <!-- make new column -->
      </td>
      <td>
      {% assign units = units | shift %} <!-- remove delimiter -->
    {% else %} <!-- make a link in the column -->
      - <a href="{{site.baseurl}}/{{units[1]}}">{{units[0]}}</a> <br/> 
      {% assign units = units | shift | shift %} <!-- remove title and url -->
    {% endif %}
  {% endfor %}
  </td>
  </tr>
  {% endfor %}

  </table>
{% endfor %}
