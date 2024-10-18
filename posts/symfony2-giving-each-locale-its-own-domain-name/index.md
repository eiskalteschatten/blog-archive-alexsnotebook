<figure><a href="https://blog.alexseifert.com/wp-content/uploads/2016/04/symfony2.svg"><img decoding="async" src="symfony2.svg" alt="Symfony"></a></figure>

After spending hours looking around online for an answer to the problem I was having setting the locale of a multi-lingual Symfony2 application based on the domain name, I finally sat down and more or less found my own solution. As is often the case, an answer to [a question on Stack Overflow](https://stackoverflow.com/questions/14648458/symfony2-detect-locale-based-on-domain-name) got me set on the right path.

*Disclaimer:* Yes, I still use Symfony2. Yes, I know Symfony3 is out. No, I want to use the LTS version. Ergo, my solution works for Symfony2 and might work for Symfony3, but I haven’t tested it. Now that that is taken care of, we can carry on.

The first thing I did was create an event listener class which I named LocalListener. Essentially, this class listens to all requests and sets the locale based on the domain. This is what the class looks like:

**LocaleListener.php**

```php
namespace AppBundle\Listeners;

use \Symfony\Component\HttpKernel\Event\GetResponseEvent;
use \Symfony\Component\HttpKernel\KernelEvents;
use \Symfony\Component\EventDispatcher\EventSubscriberInterface;
use Symfony\Component\DependencyInjection\ContainerInterface as Container;

class LocaleListener implements EventSubscriberInterface
{

    protected $domainLocales;
    protected $defaultLocale;

    public function __construct($container, $defaultLocale)
    {
        $this-&gt;domainLocales = $container-&gt;getParameter('domain_locales');
        $this-&gt;defaultLocale = $defaultLocale;
    }
    /**
     * Set default locale
     *
     * @param GetResponseEvent $event
     */
    public function onKernelRequest(GetResponseEvent $event)
    {
        $request = $event-&gt;getRequest();
        $host = $request-&gt;getHost();
        $locale = $this-&gt;defaultLocale;
        $domainLocale = array_search($host, $this-&gt;domainLocales);

        if (!empty($domainLocale)) {
            $locale = $domainLocale;
        }

        $request-&gt;setLocale($locale);
    }

    /**
     * {@inheritdoc}
     */
    static public function getSubscribedEvents()
    {
        return array(
            // must be registered before the default Locale listener
            KernelEvents::REQUEST =&gt; array(array('onKernelRequest', 17)),
        );
    }

}
```

The onKernalRequest() function checks to see if the current hostname (domain name) is in a key-value array that I’ve defined in config.yml where the key is the locale and the value is the hostname. If the hostname cannot be found, it defaults to the default locale I’ve set in the config.yml. This is what the parameters section of my config.yml looks like where I’ve defined the possible locales, the default locale (called only “locale” in the configuration) and the “domain\_locales” key-value array that I check in the event listener class above:

**config.yml**

```yaml
parameters:
    locale: en
    app.locales: en|de
    domain_locales:
          en: www.example.com
          de: www.beispiel.de
```

Once that is taken care of, we need to setup the configuration so that it also works locally in a development environment. One solution to this is the simply add the domains “www.example.com” and “www.beispiel.de” to your hosts file, but that means you will constantly have to edit the host file in order to switch between your local environment and your productive website. Symfony thankfully provides us with a file called config\_dev.yml which overrides the parameters in config.yml when working with [Symfony’s built-in server](http://symfony.com/doc/current/cookbook/web_server/built_in.html). That means you can create two new domains that will only work locally.

In order to do this, I’ve overwritten the “domain\_locals” key-value array as follows:

**config\_dev.yml:**

```yaml
parameters:
    domain_locales:
          en: symfony.localhost.com
          de: symfony.localhost.de
```

Of course, they need to be added to your hosts file as well so that your browser will know to look locally:

**Hosts file**

```
127.0.0.1       symfony.localhost.com
127.0.0.1       symfony.localhost.de
```

The last thing that needs to be done is to connect the event listener class in services.yml. You can use the following example, changing the location to match the actual location of your class:

**services.yml**

```yaml
locale_listener:
        class: YourBundle\Folder\LocaleListener
        tags:
          -  { name: kernel.event_subscriber }
        arguments: [@service_container,%locale%]
```

Now, we can start Symfony’s built-in server and browse to the English and German versions of the application:

English: http://symfony.localhost.com:8000  
German: http://symfony.localhost.de:8000

Of course, we can’t forget to add port 8000 locally. Depending on your setup, adding the port will most likely be unnecessary when accessing your application on production.

Once the application has been deployed and the server is properly configured so that both domains point to the same application (one could be setup as an alias of the other, for example), you can access your application in German or in English or whatever other locale you’ve defined via the domains you defined in config.yml.

The benefit of this setup is that you can add as many locales and domains as needed by simply changing or adding to the “domain\_locales” array so that the key matches Symfony’s locale while the value matches the domain.

**Update for Symfony 3**

Reader Joseph B. wrote a comment about how he made this code compatible with Symfony 3:

> There were some differences from Symfony 2:  
> – “parameters” was already defined in my “config.yml” file so I just needed to add the “domain\_locales” data in there.  
> – Rather than editing the “LocaleListener” class, I needed to add your changes into “LocaleSubscriber” instead.  
> – Services.yml’s syntax is slightly different; needed this instead of what was in the article:  
> AppBundle\\EventSubscriber\\LocaleSubscriber:  
> arguments: \[‘%kernel.default\_locale%’,’@service\_container’\]  
> tags: \[kernel.event\_subscriber\]