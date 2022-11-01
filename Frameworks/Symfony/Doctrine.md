[[Symfony]] [[MySQL]]

``` toc
title: "### Table of Contents"
```

### ORM
#### Table Definition
``` php
/**  
 * @ORM\Entity(repositoryClass="App\Repository\UserRepository")  
 * @ORM\Table(name="users")  
 */
 class User
 {
 }
```

#### Column Definiton
``` php
/**  
 * @ORM\Column(type="string", name="column_name")  
 */
 private string $stringCustomName;
 
/**  
 * @ORM\Column(type="string", nullable=true)  
 */
 private ?string $stringNullable;
  
/**  
 * @ORM\Column(type="integer")  
 */
 private int $integer;  
  
/**  
 * @ORM\Column(type="text")  
 */
 private string $text;  
  
/**  
 * @ORM\Column(type="datetime")  
 */
 private DateTime $dateTime;
  
/**  
 * @ORM\Column(type="boolean")  
 */
 private bool $bool;
 
/**  
 * @ORM\Column(type="float")  
 */
 private float $float;
```

#### Relations
###### One To One
``` php
/**  
 * @ORM\OneToOne(targetEntity="App\Entity\User")  
 */
 private User $user;
```

###### Many To One / One To Many
``` php
# User.php

/**  
 * @var ArrayCollection|UserRole[]  
 * @ORM\OneToMany(targetEntity="UserRole", mappedBy="user")  
 */
 private $userRoles; 
```

``` php
# UserRoles.php

/** 
 * @ORM\ManyToOne(targetEntity="App\Entity\User", inversedBy="userRoles")  
 */
 private User $user;
```

###### Many To Many
``` php
# User.php

/**  
 * @var ArrayCollection|Categories[]  
 * @ORM\ManyToMany(targetEntity="App\Entity\Category", cascade={"persist"})  
 * @ORM\JoinTable(  
 *     name="user_categories",
 *     joinColumns={@ORM\JoinColumn(name="user_id", referencedColumnName="id")},  
 *     inverseJoinColumns={@ORM\JoinColumn(name="category_id", referencedColumnName="id")}  
 * )
 */
 private $subscribedCategories;
```

``` php
# Category.php
  
/**  
 * @var ArrayCollection|User[]  
 * @ORM\ManyToMany(targetEntity="App\Entity\User", cascade={"persist"}, mappedBy="subscribedCategories")  
 */
 private $users;
```