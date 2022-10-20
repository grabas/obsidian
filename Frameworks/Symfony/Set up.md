[[Symfony]]
Documentation: https://symfony.com/doc/current/setup.html.

##### For microservices, console apps or APIs
```console
$ composer create-project symfony/skeleton:"{{version}}" my_project_directory
```

##### For monolithic web applications
```` console
$ composer create-project symfony/skeleton:"{{version}}" my_project_directory
$ cd my_project_directory
$ composer require webapp
````

## Running Symfony app
``` console
$ cd my_project_directory
$ symfony server:start
```