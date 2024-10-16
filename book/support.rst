Getting Help
============

For questions about how to use the Hub, please choose from the following resources

- Always check the :doc:`FAQ page <../guides/faq>` first

- Use the `LEAP-Pangeo Discussion Forum <https://github.com/leap-stc/leap-stc.github.io/discussions>`_ to search for previous topics or post new ones

- Join the LEAP Slack and post an issue in the `'#leap-pangeo'` channel.

- Reach out to the :ref:`support.data_compute_team` by mentioning `'@data-and-compute'` on the LEAP slack channel

- Tag `'@leap-stc/data-and-compute'` in a Github issue.


Office Hours
~~~~~~~~~~~~
We also offer in-person and virtual Office Hours on Thursdays for questions about LEAP-Pangeo.
You can reserve an appointment `here <https://app.reclaim.ai/m/leap-pangeo-office-hours>`_.

LEAP Pangeo Community Hours
~~~~~~~~~~~~~~~~~~~~~~~~~~~
We run a monthly LEAP Pangeo community event, where we'll work together on everything LEAP Pangeo! This event is open to all, and is currently in-person only (with virtual options in development). Watch out for dates in the regular LEAP emails.

If you have a problem with the hub, want to learn how to make your workflow more efficient, or just take an hour to connect with other LEAP folks and code together, bring your laptop and come by! 

This event is open to all skill levels — we aim to make it a fun, collaborative and no-stress event. We’ll have snacks, too! 

👀 **Keep an eye out for upcoming dates in the email newsletter** 🗞️

.. _support.data_compute_team:

Data and Computation Team
-------------------------

.. jinja:: team-data

  {% for person in people %}

  {{ person.name }} ({{ person.org }})
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

  {% if person.role %}
  {{ person.role }}
  {% endif %}
  {% if person.pic %}
  .. image:: images/{{ person.pic }}
     :width: 200px
     :alt: {{ person.name }}
     :class: biopic
  {% endif %}
  |
  {% if person.slack_id %}
  .. image:: https://img.shields.io/static/v1?label=&message=Slack&color=0077B5&style=flat-square&logo=slack
      :alt: Slack DM
      :target: https://leap-nsf-stc.slack.com/team/{{ person.slack_id }}
  
  {% endif %}

  {% if person.pronouns %}
  .. image:: https://img.shields.io/static/v1?label=pronouns&message={{ person.pronouns }}&color=red&style=flat-square
     :alt: Pronouns
  {% endif %}

  {% if person.github %}
  .. image:: https://img.shields.io/github/followers/{{ person.github }}?label=Github&logo=github&style=flat-square
     :alt: GitHub Profile
     :target: http://github.com/{{ person.github }}
  {% endif %}

  {% if person.twitter %}
  .. image:: https://img.shields.io/twitter/follow/{{ person.twitter }}?logo=twitter&style=flat-square
      :alt: Twitter
      :target: https://twitter.com/{{ person.twitter }}
  {% endif %}

  {% if person.linkedin %}
  .. image:: https://img.shields.io/static/v1?label=&message=LinkedIn&color=0077B5&style=flat-square&logo=linkedin
     :alt: Google Scholar
     :target: https://www.linkedin.com/in/{{ person.linkedin }}
  {% endif %}

  {% if person.website %}
  .. image:: https://img.shields.io/website?style=flat-square&url={{ person.website|urlencode }}
      :alt: Website
      :target: {{ person.website }}
  {% endif %}

  {{ person.bio }}
  {% endfor %}
