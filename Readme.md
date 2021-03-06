Three Ways To Access Symfony's Container Outside a Controller Class
=============================================================
Assuming there already exists a database set up, with a table in it, named ```persons```
```
id          name
1           John
2           Mark
```

## Using Dependency Injection

* # AppBundle/Outer/Outer.php
```
<?php

namespace AppBundle\Outer;

class Outer
{
    /**
     * @var\Doctrine\ORM\EntityManager
     */
     private $em;

     public function __construct($em)
     {
        $this->em = $em;
     }

     public function show()
     {
        return $this->em->getRepository('AppBundle:Person')->findAll();
     }
}
```

* # AppBundle/Controller/DefaultController.php
```
<?php

namespace AppBundle\Controller;

# use ...
use AppBundle\Outer\Outer;

class DefaultController extends Controller
{
    /**
     * @Route("/show")
     */
     public function showAction()
     {
        $em = $this->getDoctrine()->getManager();
        $outer = new Outer($em);
        $persons = $outer->show();
        return $this->render(..., ['persons'=>$persons]);
     }
}
```
===============================================================================================

## Extending ContainerInterface

* # AppBundle/Services/Services.php
```
<?php

namespace AppBundle\Services;

use Symfony\Component\DependencyInjection\ContainerInterface as Container;

class Services
{
    private $container;

    public function __construct(Container $container)
    {
        $this->container = $container;
    }

    public function show()
    {
        $em = $this->container->get('doctrine');
        return $em->getRepository('AppBundle:Person')->findAll();
    }
}
```

* # app/config/services.yml
```
services:
    app_services:
        class: AppBundle\Services\Services
        arguments: ["@service_container"]
```

* # AppBundle/Controller/DefaultController.php
```
<?php

namespace AppBundle\Controller;

#use ...

class DefaultController extends Controller
{
    #...
    /**
     * @Route("/show")
     */
    public function showAction()
    {
        $persons = $this->get('app_services')->show();
        return $this->render('AppBundle:Default:show.html.twig',['persons'=>$persons]);
    }
}
```

=====================================================================================================

## Extending Symfony's Controller Abstract Class

* # AppBundle/Services/Services.php
```
<?php

namespace AppBundle\Services;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;

class Services extends Controller
{
    public function show()
    {
        $em = $this->getDoctrine()->getManager();
        return $em->getRepository('AppBundle:Person')->findAll();
    }
}
```

* # app/config/services.yml
```
services:
    app_services:
        class: AppBundle\Services\Services
        calls:
          - [setContainer, ["@service_container"]]
```

* # AppBundle/Controller/DefaultController.php
```
<?php

namespace AppBundle\Controller;

#use...

class DefaultController extends Controller
{
    #...
    /**
     * @Route("/show")
     */
    public function showAction()
    {
        $persons = $this->get('app_services')->show();
        return $this->render('AppBundle:Default:show.html.twig',['persons'=>$persons]);
    }
}
```