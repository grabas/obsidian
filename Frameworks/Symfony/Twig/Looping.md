[[Twig]]

### Looping
``` twig
{% for user in users %}
	<li>{{ user.username }}</li>
{% endfor %}
```

Loop index
``` twig
{% for user in users %}
	<li>{{ loop.index }} - {{ user.username }}</li>
{% endfor %}
```

Iterating over Keys
```  twig
{% for key in users|keys %}
	<li>{{ key }}</li>
{% endfor %}
```

Keys and values
``` twig
{% for key, user in users %}
	<li>{{ key }}: {{ user.username|e }}</li>
{% endfor %}
```