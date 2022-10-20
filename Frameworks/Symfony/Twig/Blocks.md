[[Twig]]

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
