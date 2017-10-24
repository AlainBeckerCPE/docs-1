---
title: 'Naming Directory'
taxonomy:
    category:
        - docs
---

Every container running in the application server has an internal registry; we name it Naming Directory. In Java, it is called `Enterprise Naming Context` or in short `ENC`. The naming directory an object store; the container registers references to its resources. Resources can be beans or contexts provided by an application. All resources are registered in the `Naming Directory` to ensure accessability.

## Configure Directories

When the application server starts, by default, it parses the `META-INF/classes` and `WEB-INF/classes` folders of your application to find components with supported annotations.

What directories are parsed to locate annotated components can be configured in your applications configuration file. If you do not bundle a particular configuration file with your application, the default configuration is used. The default application configuration is located in `etc/appserver/conf.d/context.xml` and MUST NOT be edited. The nodes `/context/managers/manager[@name="BeanContextInterface" or @name="ServletContextInterface"]` are responsible for parsing and initializing the components.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<context
  name="globalBaseContext"
  type="AppserverIo\Appserver\Application\Application"
  xmlns="http://www.appserver.io/appserver">
  ...
  <managers>
    ...
    <manager
      name="BeanContextInterface"
      type="AppserverIo\Appserver\PersistenceContainer\BeanManager"
      factory="AppserverIo\Appserver\PersistenceContainer\BeanManagerFactory">
      <directories>
        <directory>/META-INF/classes</directory>
      </directories>
    </manager>
    <manager
      name="ServletContextInterface"
      type="AppserverIo\Appserver\ServletEngine\ServletManager"
      factory="AppserverIo\Appserver\ServletEngine\ServletManagerFactory">
      <directories>
        <directory>/WEB-INF/classes</directory>
      </directories>
    </manager>
    ...
  </managers>
</context>
```

You can bundle your application with a customized `context.xml` file. This has to be placed in your application's `META-INF` directory. The file does not have to be a full copy of the default one, it allows you to override the nodes to be customized or extended. To add a directory like `common/classes` for example, configure the `context.xml` file as follows.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<context
  name="globalBaseContext"
  type="AppserverIo\Appserver\Application\Application"
  xmlns="http://www.appserver.io/appserver">
  <managers>
    <manager
      name="BeanContextInterface"
      type="AppserverIo\Appserver\PersistenceContainer\BeanManager"
      factory="AppserverIo\Appserver\PersistenceContainer\BeanManagerFactory">
      <directories>
        <directory>/common/classes</directory>
        <directory>/META-INF/classes</directory>
      </directories>
    </manager>
  </managers>
</context>
```

> Keep in mind that the directory must be relative to your applications root directory and start with a `/`.

Detailed information about an application's configuration is available in the section [Application Configuration](../configuration#application-configuration) of the documentation.

## Register Resources

If a class is found, it is registered in the application servers naming directory in the name you have specified in the annotations `name` attribute. As the `name` attribute is optional, the bean will be registered in the naming directory with the short class name if not specified.

To inject a bean later, the developer needs to know the name it has been registered with. In the following example, the bean will be registered in the naming directory in `php:global/example/AStatelessSessionBean` whereas `example` is the name of the application.

> The name of your application is always the directory it is deployed to. The document root is per default `webapps`. For example, on a Linux system, this results in `/opt/appserver/webapps`. The name of your application is `example` and it is located in `/opt/appserver/webapps/example`.

When using annotations to inject components the completely qualified name is not neccessary, because the application server knows the context you are in. The application server tries to lookup the bean and injects it.

```php
<?php

namespace AppserverIo\Example\SessionBeans;

/**
 * @Stateless
 */
class AStatelessSessionBean
{

  /**
   * Creates and returns a new md5 hash for the passed password.
   *
   * @param string $password The password we want to hash
   *
   * @return string The md5 hash representation of the password
   */
  public function hashPassword($password)
  {
    return md5($password);
  }

  /* Creates a new user, hashes the password before.
   *
   * @param string $username The username of the user to create
   * @param string $password The password bound to the user
   *
   * @return void
   */
  public function createUser($username, $password)
  {

    // hash the password
    $hashedPassword = $this->hashPassword($password);

    /*
     * Implement functionality to create user in DB
     */
  }
}
```

## Annotations

Annotations are used to configure components. We provide several annotations that allow registering and configuring your components on the one side and injecting them during runtime on the other.

### Stateless Session Bean (@Stateless)

The `@Stateless` annotation defines a component as `Stateless` Session Bean. The annotation only supports the optional `name` attribute. If the `name` attribute is specified, the given value will be used to register the component in the `Naming Directory` instead of the short class name.

### Stateful Session Bean (@Stateful)

The `@Stateful` annotation defines a component as `Stateful` Session Bean. The annotation only supports the optional `name` attribute. If the `name` attribute is specified, the given value will be used to register the component in the `Naming Directory` instead of the short class name. The annotation has to be set at the classes DocBlock.

### Singleton Session Bean (@Singleton)

The `@Singleton` annotation defines a component as `Singleton` Session Bean. The annotation only supports the optional `name` attribute. If the `name` attribute is specified, the given value will be used to register the component in the `Naming Directory` instead of the short class name. The annotation has to be set at the classes DocBlock.

### Message-Driven Bean (@MessageDriven)

The `@MessageDriven` annotation defines a component as `Message-Driven` Bean. The annotation only supports the optional `name` attribute. If the `name` attribute is specified, the given value will be used to register the component in the `Naming Directory` instead of the short class name. The annotation has to be set at the classes DocBlock.

### Explicit Startup (@Startup)

The `@Startup` annotation configures a `Singleton` Session Bean to be initialized on application startup and can **explicitly** be used on `Singleton` Session Beans. The annotation does not accept any attributes and has to be set at the classes DocBlock.

### Post-Construct Callback (@PostConstruct)

The `@PostConstruct` annotation marks a method as `post-construct` lifecycle callback and has to be set at the methods DocBlock. The annotation can be used on all [Server-Side Component Types](../persistence-container#server-side-component-types) and does not accept any attributes.

### Pre-Destroy Callback (@PreDestroy)

The `@PreDestroy` annotation marks a method as `pre-destroy` lifecycle callback and has to be set at the methods DocBlock. The annotation can be used on all `Server-Side Component Types` and doesn't accept any attributes.

### Enterprise Beans (@EnterpriseBean)

The `@EnterpriseBean` annotation is used to inject components into other components.

The annotation can be used in two scopes. First scope is in the DocBlock of a component's class member, second scope is in the DocBlock of a class method. In both cases, the member or the method are marked as a target for `Dependency Injection`.

In the simplest case, no attribute is needed. If so, the member or parameter name MUST exactly match the components `name` that should be injected. Otherwise, you have to specify the `name` attribute and optionally the `beanName` and `beanInterface` or `lookup` attribute.

| Node Name                   | Type        | Description                                                          |
| --------------------------- | ----------- | -------------------------------------------------------------------- |
| `description`               | `string`    | Short description of the created reference.                         |
| `name`                      | `string`    | The name of the reference will be registered in the `Naming Directory`.     |
| `beanName`                  | `string`    | The `name` of the component we want to reference to.                    |
| `beanInterface`             | `string`    | The business interface we want to reference to. This has to be the `name`, suffixed with either one of `Local` or `Remote`. |
| `lookup`                    | `string`    | The fully qualified name of the component that has to be referenced to in the `Naming Directory`. |

### Resources (@Resource)

This annotation is used to inject resources into components.

As resources are classes that are initialized during application server startup to handle the main application functionality, they are not accessed by a proxy. When adding a `@Resource` annotation to inject a resource, a simple reference to the resource, using a callback, will be registered in the `Naming Directory`.

The `@Resource` annotation can be used in two scopes. First scope is in the DocBlock of a component's class member, second scope is in the DocBlock of a class method. In both cases, the member or the method are marked as a target for `Dependency Injection`.

In the simplest case, no attribute is needed. If so, the member or parameter name MUST exactly match the resource `name` that should be injected. Otherwise, you have to specify the `name` attribute and the `type` attribute.

| Node Name                   | Type        | Description                                                        |
| --------------------------- | ----------- | -------------------------------------------------------------------|
| `description`               | `string`    | Short description for the created reference.                       |
| `name`                      | `string`    | Name of the reference will be registered in the `Naming Directory`.|
| `type`                      | `string`    | The `name` of the resource we want to reference.                   |

### Persistence Unit (@PersistenceUnit)

This annotation is used to inject a [Persistence Unit](../persistence-container#persistence-unit) into components.

Similar to the resources, a real Entity Manager instance will be injected instead of a proxy.

In the simplest case, you have to specify the `unitName` attribute, which has to reference a `Persistence Unit`, you have declared in your application's `META-INF/persistence.xml` file. If you do not specify the `name` attribute the name of the member, the annotation is defined for, will be used for registration in the `Naming Directory`.

| Node Name                   | Type        | Description                                                        |
| --------------------------- | ----------- | -------------------------------------------------------------------|
| `name`                      | `string`    | Name of the reference will be registered in the `Naming Directory`.|
| `unitName`                  | `string`    | The `name` of the `Persistence Unit` we want to reference.                   |


### Example

The following example implementation of a `Singleton` session bean contains nearly all annotations and demonstrates their application.

```php
<?php

namespace AppserverIo\Example\SessionBeans;

/**
 * Example implementation of a singleton session bean that will be initialized
 * on application startup, uses post-construct and pre-destroy lifecycle callbacks
 * and dependency injection.
 *
 * @Singleton
 * @Startup
 */
class ASingletonSessionBean
{

  /**
   * The application instance that provides the entity manager.
   *
   * @var \AppserverIo\Psr\Application\ApplicationInterface
   * @Resource(name="ApplicationInterface")
   */
  protected $application;

  /**
   * A stateless session bean instance that uses property injection.
   *
   * @var \AppserverIo\Example\SessionBeans\AStatelessSessionBean
   * @EnterpriseBean
   */
  protected $aStatelessSessionBean;

  /**
   * A stateful session bean instance injected by method injection.
   *
   * @var \AppserverIo\Example\SessionBeans\AStatefulSessionBean
   */
  protected $aStatefulSessionBean;

  /**
   * Injects a stateful session bean instance.
   *
   * @param \AppserverIo\Example\SessionBeans\AStatefulSessionBean
   *
   * @return void
   * @EnterpriseBean
   */
  public function injectAStatelessSessionBean($aStatefulSessionBean)
  {
    $this->aStatefulSessionBean = $aStatefulSessionBean;
  }

  /**
   * Post-Construct lifecycle callback implementation.
   *
   * @return void
   * @PostConstruct
   */
  public function postConstruct()
  {
    // do something after initialization here
  }

  /**
   * Pre-Destroy lifecycle callback implementation.
   *
   * @return void
   * @PreDestroy
   */
  public function preDestroy()
  {
    // do something before destruction here
  }
}
```

## Deployment Descriptor

Annotations can be used for configuring almost everything. Also, it is possible to resign annotations and use an XML-based deployment descriptor called `epb.xml`. As we think that using annotations is preferred by most developers, we give a short overview of a deployment descriptors structure.

The following example is a simplified copy of the deployment descriptor of our [example](https://github.com/appserver-io-apps/example) application and provides a brief overview of the structure.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<epb xmlns="http://www.appserver.io/appserver">
  <enterprise-beans>
    <session>
      <session-type>Singleton</session-type>
      <epb-name>ASingletonProcessor</epb-name>
      <epb-class>AppserverIo\Apps\Example\Services\ASingletonProcessor</epb-class>
      <init-on-startup>true</init-on-startup>
      <post-construct>
        <lifecycle-callback-method>initialize</lifecycle-callback-method>
      </post-construct>
    </session>
    <session>
      <session-type>Stateful</session-type>
      <epb-name>UserProcessor</epb-name>
      <epb-class>AppserverIo\Apps\Example\Services\UserProcessor</epb-class>
      <pre-destroy>
        <lifecycle-callback-method>destroy</lifecycle-callback-method>
      </pre-destroy>
    </session>
    <session>
      <session-type>Stateless</session-type>
      <epb-name>SampleProcessor</epb-name>
      <epb-class>AppserverIo\Apps\Example\Services\SampleProcessor</epb-class>
      <post-construct>
        <lifecycle-callback-method>initialize</lifecycle-callback-method>
      </post-construct>
      <pre-destroy>
        <lifecycle-callback-method>destroy</lifecycle-callback-method>
      </pre-destroy>
      <epb-ref>
        <epb-ref-name>UserProcessor</epb-ref-name>
        <lookup-name>php:global/example/UserProcessor</lookup-name>
        <injection-target>
          <injection-target-class>
	   AppserverIo\Apps\Example\Services\SampleProcessor
          </injection-target-class>
          <injection-target-property>userProcessor</injection-target-property>
        </injection-target>
      </epb-ref>
      <res-ref>
        <description>Reference to the application</description>
        <res-ref-name>ApplicationInterface</res-ref-name>
        <injection-target>
          <injection-target-class>
            AppserverIo\Apps\Example\Services\AbstractProcessor
          </injection-target-class>
          <injection-target-method>injectApplication</injection-target-method>
        </injection-target>
      </res-ref>
    </session>
  </enterprise-beans>
</epb>
```

The structure is self-explanatory, as it reflects the annotations `@EnterpriseBean` and `Resource`. The following table describes all possible nodes and their meanings.

`/epb/enterprise-beans/session`
either defines a `SLSB`, `SFSB` or a `SSB`.

| Node Name                   | Type        | Description                                                        |
| --------------------------- | ----------- | -------------------------------------------------------------------|
| `session-type`              | `string`    | Can be `Stateless`, `Stateful` or `Singleton`.                     |
| `epb-name`                  | `string`    | Short name of the component used for registration in naming directory. |
| `epb-class`                 | `string`    | Fully qualified class name of the component's class.               |
| `init-on-startup`           | `boolean`   | `true` if the component should be instanciated on application startup. This can only be set to `true` if `session-type` is `Singleton`. |

`/epb/enterprise-beans/message-driven`
defines a `MDB`.

| Node Name                   | Type        | Description                                                        |
| --------------------------- | ----------- | -------------------------------------------------------------------|
| `epb-name`                  | `string`    | Short name of the component used for registration in naming directory. |
| `epb-class`                 | `string`    | Fully qualified class name of the component's class.               |

`/epb/enterprise-beans/[session or message-driven]/post-construct`
adds a `post-construct` lifecycle callback to the component.

| Node Name                   | Type        | Description                                                          |
| --------------------------- | ----------- | -------------------------------------------------------------------- |
| `lifecycle-callback-method` | `string`    | Name of a class method that will be invoked after the component been initialized. |

`/epb/enterprise-beans/[session or message-driven]/pre-destroy`
adds a `pre-destroy` lifecycle callback to the component.

| Node Name                   | Type        | Description                                                        |
| --------------------------- | ----------- | -------------------------------------------------------------------|
| `lifecycle-callback-method` | `string`    | Name of a class methode that will be invoked before the class will be destroyed. |

`/epb/enterprise-beans/[session or message-driven]/epb-ref`
creates a reference to the remote or local business interface of the defined session bean in the naming directory in `php:global/example/env/[epb-ref-name][Local or Remote]`. This reference can be used by other components or for Dependency Injection purposes.

| Node Name                   | Type        | Description                                                        |
| --------------------------- | ----------- | -------------------------------------------------------------------|
| `description`               | `string`    | A short description of the reference that will be created.         |
| `epb-ref-name`              | `string`    | The name of the reference created in the naming directory.         |
| `epb-link`                  | `string`    | Name of referenced component. This name is by default the short class name or can be overwritten by the `name` attribute of the `Stateless`, `Stateful` or `Singleton` annotations. |
| `lookup-name`               | `string`    | Optionally to the `epb-link` this value contains the fully qualified name of the referenced component. |
| `remote`                    | `boolean`   | If a value has been specified, a reference to the remote proxy will be created instead of a local one. |

`/epb/enterprise-beans/[session or message-driven]/res-ref`
creates a reference to the defined resource in the naming directory in `php:global/example/env/[res-ref-name]`. This reference can be used by other components or for Dependency Injection purposes.

| Node Name                   | Type        | Description                                                        |
| --------------------------- | ----------- | -------------------------------------------------------------------|
| `description`               | `string`    | A short description of the reference that will be created.         |
| `res-ref-name`              | `string`    | The name of the reference created in the naming directory.         |
| `res-ref-type`              | `string`    | The type of the reference resource.                                |

`/epb/enterprise-beans/[session or message-driven]/[ebp-ref or res-ref]/injection-target`
injects the reference by either using the method or property defined. The class name is of interest, if there is a hierarchy and the target class has to be specified explictly.

| Node Name                   | Type        | Description                                                        |
| --------------------------- | ----------- | -------------------------------------------------------------------|
| `injection-target-class`    | `string`    | The class we want to inject the reference.                         |
| `injection-target-method`   | `string`    | Use this method to inject the reference on runtime.                |
| `injection-target-property` | `string`    | Inject the reference to this property, whereas this node or `injection-target-method` can be specified. |

> Annotations are default values, whereas a deployment descriptor enables a developer or a system administrator to override values specified in annotations. So keep in mind, that a deployment descriptor will always override the values specified by annotations.