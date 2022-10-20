[[Symfony]]
Documentation: https://twig.symfony.com/doc

``` toc
title: "## Table of Contents"
```

Standalone HTML templating framework by Symfony.

#### Extends
extends targeted document by filling out "block" blocks 
``` twig
{% extends 'base.html.twig' %}
```

#### Blocks 
blocks to fill in the extended template
``` twig
# ../base.html.twig
{% block title %}{% endblock %}
{% block css %}{% endblock %}
{% block body %}{% endblock %}
{% block scripts %}{% endblock %}
```

``` twig
# ../user_detail.html.twig
{% extends 'base.html.twig' %}

{% block title %}  
   User Detail
{% endblock %}

{% block css %}
	<style>
		...
		...
		...
	</style>
{% endblock %}

{% block body %}
	<div class="...">
		...
		...
		...
	</div>
{% endblock %}

{% block scripts %}
	<script>
		$(document).ready(function(){
			...
			...
			...
		})
	</script>
{% endblock %}

```

### Passing and Accessing variables
``` php
// ../UserController.php
public function detailAction(string $id)  
{  
    $user = $this->userService->getById($id);  
  
    return $this->render("user.html.twig", [  
        'user' => $user
    ]);  
}
```

``` twig
# user.html.twig
<li>{{ user.username }}</li>
```

### Looping

```twig
{% for user in users %}
	<li>{{ user.username }}</li>
{% endfor %}
```

##### Loop index

```twig
{% for user in users %}
	<li>{{ loop.index }} - {{ user.username }}</li>
{% endfor %}
```

##### Iterating over Keys

```twig
{% for key in users|keys %}
	<li>{{ key }}</li>
{% endfor %}
```

##### Keys and values

```twig
{% for key, user in users %}
	<li>{{ key }}: {{ user.username|e }}</li>
{% endfor %}
```

## Conditions

```  twig
{% if user.isRegistered %}
	<p>{{user.userName}}</p>
{% else %}
	<p>Anonymous</p>
{% endif %}
```

##### Variations

``` twig
{% if user.isRegistered == false %}
{% if not user.isRegistered %}
{% if user.age > 21 and user.age < 65 %}
{% if user.email is not null %}
```
