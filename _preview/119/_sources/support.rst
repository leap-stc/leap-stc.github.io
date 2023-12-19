Getting Help
============

For questions about how to use the Hub, please use the LEAP-Pangeo discussion forum:

- `LEAP-Pangeo Discussion Forum <https://github.com/leap-stc/leap-stc.github.io/discussions>`_

Office Hours
~~~~~~~~~~~~
We also offer in-person and virtual Office Hours on Thursdays for questions about LEAP-Pangeo.
You can reserve an appointment `here <https://app.reclaim.ai/m/leap-pangeo-office-hours>`_.

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

    {% if person.slack_id %}
    .. image:: https://img.shields.io/static/v1?label=&message=Slack&color=0077B5&style=flat-square&logo=slack
        :alt: Slack DM
        :target: https://leap-nsf-stc.slack.com/team/{{ person.slack_id }}
    
    {% endif %}

    {% if person.pic %}
    .. image:: images/{{ person.pic }}
       :width: 200px
       :alt: {{ person.name }}
       :class: biopic
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
