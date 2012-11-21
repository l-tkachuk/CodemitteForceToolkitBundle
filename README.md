Introduction
============
CodemitteForceToolkitBundle is a [Symfony 2] bundle to integrate Force.com Toolkit for PHP 5.3 into your [Symfony 2] project. Simply enable the bundle, configure SOAP access to your salesforce organisation and start working.

THIS BUNDLE AND IT'S LIBRARY DEPENDENCIES IS STILL SUBJECT OF HEAVY DEVELOPMENT AND SHOULD NOT BY CONSIDERED STABLE! DONT USE IT IN PRODUCTION UNTIL YOU REALLY NOW WHAT YOU'RE DOING (though i don't in every circumstances.)

This bundle is part of a multi-tenant self service portal application currently developed by the company i work for and will recieve updates/backports from time-to-time, but there is no "official release schedule".

In the meanwhile, some other, rather popular libraries arised, perhaps you want to take a look at these ones:

* [SalesforceClientBundle]
* [SalesforceMapperBundle]

Features
========
* SOAP client abstraction layer through [CodemitteSoap library]
  * SOAP API: partner.wsdl.xml, enterprise.wsdl.xml
  * Metadata API: Use .describeX()-Methods to fetch metadata information
  * Basis soap interface for custom webservices (APEX-Webservice classes)
* SOQL query abstraction layer through [CodemitteForceToolkit]
  * Build queries "by hand"
  * Build queries by using the fluent API of a SOQL query builder
  * Chain both string queries and API-built queries
  * Use limit(), offset(), orderBy(), where() methods to easily implement filters
* Symfony 2 form component integration
  * Validators and form types/widgets for each relevant salesforce datatype is provided, e.g. picklists.
    * Easily handle i18n and l10n by simply changing the locale of the underlying [Force.com] API user.
    * Handle picklist entry display by modifying your API user's profile, or object record types.
    * Handle currency or date fields only dependent on [Force.com] settings
* API user factory
  * Use different API users dependent on the current webuser's locale

Installation
=============

By composer/packagist (recommended)
-----------------------------------

1. Add the following line to your composer.json file into the "require"-section:
   
   ```javascript
   require {
     [...]
     "codemitte/ForceToolkitBundle" : "dev-master"
     [...]
   }
   ```

2. After that run the composer update command:

   ```bash
   $ php ./composer.phar update
   ```

   If you face problems, refer to the official [composer documentation]. Remark the required whitelist entry when using .phar-files with a php executable that has ext/suhosin enabled!

3. Enable the freshly downloaded bundle to your Kernel PHP file (usually to find in app/AppKernel.php):

   ```php
   class AppKernel extends Kernel
   {
     public function registerBundles()
     {
       return array(
         [...]
         new \Codemitte\Bundle\ForceToolkitBundle\CodemitteForceToolkitBundle()
       );
     }
   }
   ```

4. Clear all caches

Manual installation
-------------------

1. Download/clone the bundle sources into your project (recommended: Define the github repository as a [git submodule])
   The bundle should reside in an updateable vendor directory, the location should reflect the full namespaced path, e.g.
   ``` 
   vendor/codemitte/codemitte-force-toolkit-bundle/Codemitte/Bundle/ForceToolkit/{repository root, the dir where the .git directory resides}
   ```
2. Download/clone the bundle's dependencies as well
   - [CodemitteSoap]
   - [CodemitteForceToolkit]
   - [CodemitteCommon]
3. Add the classpaths of the bundle and all dependencies to your autoload.php file. The mapping is
   ```php
      'Codemitte\\Bundle\\ForceToolkitBundle' => 'path/to/cloned/repository/'
   ```

   for instance:
   
   ```php
      'Codemitte\\Bundle\\ForceToolkitBundle' => 'vendor/codemitte/codemitte-force-toolkit-bundle',
      'Codemitte\\ForceToolkit' => 'vendor/codemitte/codemitte-force-toolkit',
      'Codemitte\\Soap' => 'vendor/codemitte/codemitte-soap',
      'Codemitte\\Common' => 'vendor/codemitte/codemitte-common'
   ```
4. Enable the freshly downloaded bundle to your Kernel PHP file (usually to find in app/AppKernel.php):
   
   ```php
   class AppKernel extends Kernel
   {
     public function registerBundles()
     {
       return array(
         [...]
         new \Codemitte\Bundle\ForceToolkitBundle\CodemitteForceToolkitBundle()
       );
     }
   }
  ```

5. Clear all caches
   
Configuration
=============
The bundle provides a Configuration class which defines the schema of all config keys you may use in your config.yml (xml|php) files, as usual. The full schema is:

```yaml
codemitte_force_toolkit:
  soap_api_client:
    classname: "Codemitte\\ForceToolkit\\Soap\\Client\\PartnerClient"
    connection_ttl: 28800
    service_location: ~
    wsdl_location: "%kernel.root_dir%/config/wsdl/prod/partner.wsdl.xml"
    api_users:
      default: { username: "user1@myorg.tld", password: "********"  }
      locales:
        en: { username: "user1@myorg.co.uk", password: "********"  }
        de: { username: "user1@myorg.de", password: "********"  }
        de_AT: { username: "user1@myorg.at", password: "********"  }
  metadata:
    cache_service_id: "codemitte_forcetk.metadata.file_cache"
    cache_location: "%kernel.root_dir%/cache/forcetk"
    cache_ttl: -1
``` 

A minimal configuration may look like this:

```yaml
codemitte_force_toolkit:
  soap_api_client:
    wsdl_location: "%kernel.root_dir%/config/wsdl/prod/partner.wsdl.xml"
    api_users:
      default: { username: "user1@myorg.co.uk", password: "********"  }
      locales:
        en: { username: "user1@myorg.co.uk", password: "********"  }
  metadata:
    cache_service_id: "codemitte_forcetk.metadata.array_cache"
``` 
  
Configuration options explained
-------------------------------
* soap_api_client.classname: 
  The classname of the client PHP class, usually 
  * "Codemitte\\ForceToolkit\\Soap\\Client\\PartnerClient"
  * "Codemitte\\ForceToolkit\\Soap\\Client\\EnterpriseClient"
  Future plans tend to implemend a factory that is able to auto-detect the correct client class from the given WSDL information.
* soap_api_client.connection_ttl:
  The time-to-live of an API user, stored in memory (APC, for instance, is the only adapter for connection storage at the moment). Since a API connection is reset each 24 h, the ttl should reflect this value. If an unexpected connection error occurs, an symfony 2 exception event handler will re-bootstrap the API connection gracefully.
* soap_api_client.service_location:
  Use this flag to override the service location endpoint defined in the underlying WSDL file. This is useful when e.g. using a single WSDL file in different development environments (A production WSDL against a Quality Assurance sandbox, for instance). Though, best practice seems to maintain dedicated .wsdl-files for each organisation to avoid strange "could not connect to host" issues.
* soap_api_client.api_users:
  Array to define one or more API users for different frontend user locales. See example above. Each user must have API usage permission within salesforce.
* metadata.cache_service_id: 
  The cache service id for the metadata cache (metadata are cleaned-up results from describeSobject()/describeLayout() calls.) Available values are:
  * "codemitte_forcetk.metadata.file_cache": for production 
  * "codemitte_forcetk.metadata.array_cache": for development
* metadata.cache_location: 
  The cache location (currently in file system only) -- refactoring candidate!!
* metadata.cache_ttl: The lifetime of the metadata cache until it is about to invalidate. -1 means the cache does never expire.                                                     

Documentation
=============

Basic usage of the client/query builder
---------------------------------------

src/AcmeBundle/Controller/DefaultController.php
```php
/** @var $client Codemitte\\ForceToolkit\\Soap\\Client\\PartnerClient
$client = $this->get('codemitte_forcetk.client');
/** @var $accounts Codemitte
$response = $client->query('SELECT Id, Name FROM Account LIMIT 10');
foreach($response['result']['records'] AS $account)
{
    print_r($account);
}
```
$response is Instanceof \Codemitte\Soap\Mapping\GenericResult. You may access each property by array notation, object notation of method notation:

```php
$response['result']
$response->result
$response->get('result')
```
$response->get('result')->get('records'); is instanceof Codemitte\Soap\Mapping\GenericResultCollection which adds methods for easy traversal.
```php
foreach($response['result']['records'] AS $account)
{
    print_r($account);
}
```
(Please refer to the client interface (Codemitte\ForceToolkit\Soap\Client\APIInterface) for a list of all available service call methods.)

$client->query() is a low-level API method for fireing SOQL queries against the platform. A more convinient way is to utilize the QueryBuilder:

src/AcmeBundle/Controller/AccountController.php
```php
public function showAccountAction($id)
{
  /** @var $queryBuilder Codemitte\ForceToolkit\Soql\Builder\QueryBuilder
  $queryBuilder = $this->get('codemitte_forcetk.query_builder');
  $account = $queryBuilder->prepareStatement('SELECT Id, Name FROM Account WHERE Id = :id')->bind(array(
    'id' => $id
  ))->fetchOne();
  if(null === $account)
  {
    throw new NotFoundHttpException(sprintf('The account with Id "%s" could not be found', $id));
  } 
  return $this->render('AcmeBundle:Account:show.html.twig', array(
    'account' => $account
  );
}
```

Analogously, the account list view:

src/AcmeBundle/Controller/AccountController.php
```php
public function showAccountAction($limit = 20, $offset = 0, $orderBy = 'Name')
{
  /** @var $queryBuilder Codemitte\ForceToolkit\Soql\Builder\QueryBuilder
  $queryBuilder = $this->get('codemitte_forcetk.query_builder');
  
  // MAXIMUM OFFSET IS 2000!!!
  $accounts = $queryBuilder->prepareStatement('SELECT Id, Name FROM Account')->orderBy($orderBy)->limit($limit)->offset($offset)->fetch();
  return $this->render('AcmeBundle:Account:show.html.twig', array(
    'accounts' => $accounts
  );
}
```
Please refer to the query builder interface for further information.

The form component
------------------
[... more to come ... ]

  [Force.com]: http://force.com
  [CodemitteForceToolkit]: https://github.com/joshiausdemwald/Force.com-Toolkit-for-PHP-5.3
  [CodemitteSoap library]: https://github.com/joshiausdemwald/CodemitteSoap
  [CodemitteSoap]: https://github.com/joshiausdemwald/CodemitteSoap
  [CodemitteCommon]: https://github.com/joshiausdemwald/CodemitteCommon
  [SalesforceClientBundle]: https://packagist.org/packages/ddeboer/salesforce-client-bundle
  [SalesforceMapperBundle]: https://packagist.org/packages/ddeboer/salesforce-mapper-bundle
  [composer documentation]: http://getcomposer.org/
  [git submodule]: http://git-scm.com/book/en/Git-Tools-Submodules
  [Symfony 2]: http://symfony.com/
    