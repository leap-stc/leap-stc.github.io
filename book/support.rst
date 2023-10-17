Support
=======

.. jinja:: team-data

    {% for person in people %}
    {{person.name}}
    ~~~~~
    {{person.role}}
    
    {% endfor %}