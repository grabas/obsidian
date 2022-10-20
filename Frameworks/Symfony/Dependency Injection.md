[[Symfony]]
Documentation: https://symfony.com/doc/current/components/dependency_injection.html

**Standard way to instantiate objects and handle dependency management in your PHP applications. The heart of the DependencyInjection component is a container, which holds all the available services in the application.**

During the bootstrapping phase of your application, you're supposed to register all services in your application into the container. At a later stage, a container is responsible for creating services as needed. More importantly, a container is also responsible for creating and injecting dependencies of the services.

The benefit of this approach is that you don't have to hard code the process of instantiating objects, since dependencies will be detected and injected automatically. This creates a loose coupling between parts of your application.

In this article, we'll explore how you can unleash the power of the DependencyInjection component. As usual, we'll start with installation and configuration instructions, and we'll implement a few real-world examples to demonstrate the key concepts.