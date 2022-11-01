[[Twig]]

## Conditions

```  twig
{% if user.isRegistered %}
	<p>{{user.userName}}</p>
{% else %}
	<p>Anonymous</p>
{% endif %}
```

Variations

``` twig
{% if user.isRegistered == false %}
{% if user.email is defined %}
{% if not user.isRegistered %}
{% if user.age > 21 and user.age < 65 %}
{% if user.email is not null %}
```

