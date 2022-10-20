[[Twig]]

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
