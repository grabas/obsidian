[[Design Patterns]] [[Symfony]]

In Symfony
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
        ...
        ...
        ...  
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

and now register the CustomCompilerPass to the Symfony Kernel

``` php
# App/Kernel.php  
class Kernel extends BaseKernel  
{  
    use MicroKernelTrait;  
  
    protected function build(ContainerBuilder $container): void  
    {  
        $container->addCompilerPass(new CustomCompilerPass());  
    }
    ...
    ...
    ...
)    
```