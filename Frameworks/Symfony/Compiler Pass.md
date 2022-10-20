
``` toc
title: "## Table of Contents"
```

### Overview
Compiler passes give you an opportunity to manipulate other [service definitions](https://symfony.com/doc/current/service_container/definitions.html) that have been registered with the service container.

This comes in handy when you need to process something in multiple ways or when you have multiple ways to process something. Says pretty much nothing, right? Let me give you some examples.

### Example 1 ("Multiple ways to process something")

One common example is user sigh ins. Nowdays you can log into an app via multiple providers like Facebook, Google, Apple and many others. The process authentication and getting/creating an user is in principle the same but the concrete implementation will differ from provider to provider.

##### Remote User Provider Interface
You can define interface like this.

``` php
interface RemoteUserProviderInterface
{    
	public function getUser(LoginRequest $request): User;
	public function supports(LoginRequest $request): bool;
}
```

##### Remote User Provider Class
And a class that implements said interface. (We will have a class for every provider we want to implement into our application). Let's use Facebook for this example.

The _**getUser(...)**_ method handles the authentification of identity token against Facebook oAuth Server. If the authentification succeeds we can fetch user from our database (or create a new one if we don't find any). 

The **_supports(...)_** method checks if the login was request via Facebook provider.

``` php
  
class FacebookUserProviderFacade implements RemoteUserProviderInterface  
{  
	private UserLoginFacade $userLoginFacade;  
    private FacebookClient $facebookClient;
  
    public function __construct(  
        FacebookClient $facebookClient,  
        UserLoginFacade $userLoginFacade
	) {  
		$this->userLoginFacade = $userLoginFacade;  
        $this->facebookClient = $facebookClient;
    }  
    
	public function getUser(LoginRequest $request): User  
	{  
		$identityToken = $request->getPassword();  
		
		$graphUser = $this->facebookClient->findUserByToken($identityToken);  
		$user = $this->userLoginFacade->findUserByFacebookGraphUser($graphUser);  
		
		if ($user === null) {  
			$user = $this->userLoginFacade->registerNewUser(graphUser);
		}
		
		return $user;  
	}
	 
	public function supports(LoginRequest $request): bool  
	{  
		return $request->getProvider() === RemoteUserProvidersEnum::FACEBOOK;  
	}
}
```

##### Remote User Provider Facade
Now we need some sort of facade that will loop through every instance of **RemoteUserProviderInterface** and executes the _**getUser() **_ method on matched provider.

``` php
class RemoteUserProviderFacade  
{  
	/** @var RemoteUserProviderInterface[] */  
	private array $remoteUserProviders = [];  
  
    public function resolveUser(LoginRequest $request): User  
    {  
		foreach ($this->remoteUserProviders as $userProvider) {  
            if ($userProvider->supports($request)) {  
	            return $userProvider->getUser($request);  
			}
		}
		
		throw new InvalidProviderException();
    }
    
	public function addUserProvider(RemoteUserProviderInterface $userProvider)  
	{        
		$this->remoteUserProviders[] = $userProvider;  
	}
}
```

##### Service Definitions
Noticed the **addUserProvider(...) ** method? This is where the **Compiler Pass** comes to the scene.
In order to pass every instace of **RemoteUserProviderInterface** we need tag them in **_service.yaml_**. We could tag every instance by hand but luckily we can use the _\_**instanceof**_ keyword that will do that for us... and automatically as our remote providers grow.

``` yaml
#  services.yaml
_instanceof:  
    App\...\...\...\RemoteUserProviderInterface:  
    tags:  
        - { name: 'app.remote_user_provider_tag' }
```

##### Compiler Pass
Now we need to specify the way in which Symfony will handle the [[Dependency Injection]] for **RemoteUserProviderFacade**.

``` php
class RemoteUserProviderCompilerPass implements CompilerPassInterface  
{  
    public function process(ContainerBuilder $container)  
    {        
	    if (!$container->has(RemoteUserProviderFacade::class)) {  
            return;  
        }
	    
	    // Get the facade
        $facade = $container->findDefinition(RemoteUserProviderFacade::class);  

		// Get tagged services (every instance of RemoteUserProviderInterface)
        $taggedServices = $container->findTaggedServiceIds('app.remote_user_provider_tag');  

		// Add every tagged service to the facade via adder method
        foreach ($taggedServices as $id => $tags) {  
			$facade->addMethodCall('addUserProvider', [  
	            new Reference($id)  
	        ]);
		}    
	}
}
```

##### Registering Compiler Pass
Now we need to pass this DI definition to our framework.

``` php
# App/Kernel.php  
class Kernel extends BaseKernel  
{  
    use MicroKernelTrait;  
  
    protected function build(ContainerBuilder $container): void  
    {  
        $container->addCompilerPass(new RemoteUserProviderCompilerPass());  
    }    
```

And voila! Seems to be quite a lot of steps for a complete setup but now every time we want to add a new third party service for users to log in with (Outlook, Office365, GitHub etc.) it's as simple as creating a new class implementing RemoteUserProviderInteface and defining the specifics... the framework will take care of the rest.

### Example 2 ("Processing something in multiple ways")

For this example imagine you have a music app. This app naturally has a media player UI user interacts with and we would naturally want to track and log those interactions for recommendation engine, statistics, history etc.

We would have to define some interaction types like Play, Pause, Skip, Stop, Seek and so on.
One solution to process these interactions would look something like this. (Purposely ineefective)

``` php
	switch ($inteaction->getType()) {
		case InteractionTypeEnum::PLAY:
			$this->logRepository->store($inteaction);
			$this->historyFacade->addTrackToUserHistory($track, $user);
			$this->statisticsFacade->incrementTrackPlayCount($track);
			$this->recommenderFacade->sendToRecommender($interaction);
			break;
		case InteractionTypeEnum::STOP:
			$this->logRepository->store($inteaction);
			$this->statisticsFacade->incrementTrackStopCount($track);
			$this->recommenderFacade->sendToRecommender($interaction);
			break;
		case InteractionTypeEnum::SKIP:
			$this->logRepository->store($inteaction);
			$this->statisticsFacade->incrementTrackSkipCount($track);
			$this->recommenderFacade->sendToRecommender($interaction);
			break;
		case InteractionTypeEnum::SEEK:
			$this->logRepository->store($inteaction);
			break;
		case InteractionTypeEnum::FINISHED:
			$this->recommenderFacade->sendToRecommender($interaction);
			break;
		default:
			throw new InvalidRequestException("Interaction type not valid");
	}
```

You can see planty of problems with this solution: Readability, introducting new types of interactions would bloat the code substantially, debuging would be a nightmare, making changes or adding new ways to process interactions would be painful and code breaking changes would affect more than asked for. 

Compiler Pass to the rescue! The same story.

##### Interface
Let's define an interface
``` php
interface InteractionProcessInterface  
{  
    public function processInteraction(InteractionDto $interactionDto): void;  
    public function supports(InteractionDto $interactionDto): bool;  
}
```

##### Classes
And class for a single process! 

``` php
class InteractionStoreToDatabaseProcess implements InteractionProcessInterface  
{  
    public function processInteraction(InteractionDto $interactionDto): void  
    {  
        // Store to database  
    }
    
    public function supports(InteractionDto $interactionDto): bool  
    {  
	    $type = $interactionDto->getType();
        return InteractionTypeEnum::isDatabaseType($type);
    }
}
```

and 

``` php
class InteractionSendToRecommenderProcess implements InteractionProcessInterface  
{  
    public function processInteraction(InteractionDto $interactionDto): void  
    {  
        // Send to recommender 
    }
    
    public function supports(InteractionDto $interactionDto): bool  
    {  
	    $type = $interactionDto->getType();
		return InteractionTypeEnum::isRecommenderType(type);  
    }
}
```

and

``` php
class UserHistoryProcess implements InteractionProcessInterface  
{  
    public function processInteraction(InteractionDto $interactionDto): void  
    {  
        // Update user history
    }
    
    public function supports(InteractionDto $interactionDto): bool  
    {  
	    $type = $interactionDto->getType();
		return InteractionTypeEnum::isHistoryRelevant(type);  
    }
}
```

and whatever you want, need or can imagine :) Statistics like play, unique users, finished tracks, track retension et cetera ad infinitum ad nauseam

##### Facade

``` php
class InteractionTypeProcessor  
{  
    /** @var InteractionProcessInterface[] */  
    private array $interactionProcesses = [];
    
    public function processInteraction(InteractionDto $interactionDto): void  
    {  
        /** @var InteractionProcessInterface $process */  
        foreach ($this->interactionProcesses as $process) {  
            if ($process->supports($interactionDto)) {
	            try {
		            $process->processInteraction($interactionDto)
	            } catch (Throwable $excetion) {
		            // Log
	            }
            }
		 }
	 }

    public function addInteractionPrecess(InteractionProcessInterface $interactionProcessor)  
    {
	    $this->interactionProcesses[] = $interactionProcessor;  
    }
}
```

##### Service Definition

``` yaml
#  services.yaml
_instanceof:  
    App\Component\...\Interfaces\InteractionProcessInterface:  
    tags:  
        - { name: 'app.interaction_process_interface_tag' }
```

##### Compiler Pass

``` php
class InteractionCompilerPass implements CompilerPassInterface  
{  
    public function process(ContainerBuilder $container)  
    {   
	    if (!$container->has(InteractionTypeProcessor::class)) {  
            return;  
        }
        
         // Get the facade
        $facade = $container->findDefinition(InteractionTypeProcessor::class);  

		// Get tagged services
        $taggedServices = $container->findTaggedServiceIds('app.interaction_process_interface_tag');  

		// Add every tagged service to the facade via adder method
        foreach ($taggedServices as $id => $tags) {  
			$facade->addMethodCall('addInteractionPrecess', [  
	            new Reference($id)  
	        ]);
		}    
	 }
 }
```

##### Register

``` php
# App/Kernel.php  
class Kernel extends BaseKernel  
{  
    use MicroKernelTrait;  
  
    protected function build(ContainerBuilder $container): void  
    {  
        $container->addCompilerPass(new InteractionCompilerPass());  
    }    
```

Now we have a flexible and easily expandable solution. To add a new process just define it and specify which type it accepts under the interface. Best paired with RabbitMQ Consumer so the interactions are queued properly :) 

### Template

``` php
interface CustomProcessInterface  
{  
    public function process(CustomDto $customDto): void;  
    public function supports(CustomDto $customDto): bool;  
}
```

``` php
  
class CustomProcess implements CustomProcessInterface  
{  
    public function process(CustomDto $customDto): void  
    {  
        // Code  
    }
    
    public function supports(CustomDto $customDto): bool  
    {  
        return $customDto->getType() === CustomDtoEnum::WHATEVER
    }
}
```

``` yaml
#  services.yaml
_instanceof:  
    App\Component\...\Interfaces\CustomProcessInterface:  
    tags:  
        - { name: 'app.custom_process_interface_tag' }
```

``` php
class CustomProcessor  
{  
    /** @var CustomProcessInterface[] */  
    private array $customProcesses;
  
    public function __construct()  
    {
	    $this->customProcesses = [];  
    }
    
    public function processCustomDto(CustomDto $customDto): void  
    {  
        /** @var CustomProcessInterface $process */  
        foreach ($this->customProcesses as $process) {  
            if ($process->supports($customDto)) {  
                 $process->process($customDto)
			}
		}
	}
	
    public function addProcess(CustomProcessInterface $customInterface)  
    {        
	    $this->customProcesses[] = $customInterface;  
    }
}
```

``` php
# App/CompilerPass/CustomCompilerPass.php

class CustomCompilerPass implements CompilerPassInterface  
{  
    public function process(ContainerBuilder $container)  
    {   
	    if (!$container->has(CustomProcessor::class)) {  
            return;  
        }
        
        $definition = $container->findDefinition(CustomProcessor::class);  
        $taggedServices = $container->findTaggedServiceIds('app.custom_process_interface_tag');  
  
        foreach ($taggedServices as $id => $tags) {  
	        $definition->addMethodCall('addProcess', [  
				new Reference($id)  
			]);
		}
	 }
 }
```

``` php
# App/Kernel.php  
class Kernel extends BaseKernel  
{  
    use MicroKernelTrait;  
  
    protected function build(ContainerBuilder $container): void  
    {  
        $container->addCompilerPass(new CustomCompilerPass());  
    }    
```