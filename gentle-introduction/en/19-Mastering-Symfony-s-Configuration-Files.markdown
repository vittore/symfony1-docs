Chapter 19 - Mastering Symfony's Configuration Files
====================================================

Now that you know symfony very well, you are already able to dig into its code to understand its core design and discover new hidden abilities. But before extending the symfony classes to match your own requirements, you should take a closer look at some of the configuration files. Many features are already built into symfony and can be activated by just changing configuration settings. This means that you can tweak the symfony core behavior without overriding its classes. This chapter takes you deep into the configuration files and their powerful capabilities.

Symfony Settings
----------------

The `frontend/config/settings.yml` file contains the main symfony configuration for the `frontend` application. You have already seen the function of many settings from this file in the previous chapters, but let's revisit them.

As explained in Chapter 5, this file is environment-dependent, which means that each setting can take a different value for each environment. Remember that each parameter defined in this file is accessible from inside the PHP code via the `sfConfig` class. The parameter name is the setting name prefixed with `sf_`. For instance, if you want to get the value of the `cache` parameter, you just need to call `sfConfig::get('sf_cache')`.

### Default Modules and Actions

Symfony provides default pages for special situations. In the case of a routing error, symfony executes an action of the `default` module, which is stored in the `sfConfig::get('sf_symfony_lib_dir')/controller/default/` directory. The `settings.yml` file defines which action is executed depending on the error:

  * `error_404_module` and `error_404_action`: Action called when the URL entered by the user doesn't match any route or when an `sfError404Exception` occurs. The default value is `default/error404`.
  * `login_module` and `login_action`: Action called when a nonauthenticated user tries to access a page defined as `secure` in `security.yml` (see Chapter 6 for details). The default value is `default/login`.
  * `secure_module` and `secure_action`: Action called when a user doesn't have the credentials required for an action. The default value is `default/secure`.
  * `module_disabled_module` and `module_disabled_action`: Action called when a user requests a module declared as disabled in `module.yml`. The default value is `default/disabled`.

Before deploying an application to production, you should customize these actions, because the `default` module templates include the symfony logo on the page. See Figure 19-1 for a screenshot of one of these pages, the error 404 page.

Figure 19-1 - Default 404 error page

![Default 404 error page](http://www.symfony-project.org/images/book/1_4/F1901.jpg "Default 404 error page")

You can override the default pages in two ways:

  * You can create your own default module in the application's `modules/` directory, override all the actions defined in the `settings.yml` file (`index`, `error404`, `login`, `secure`, `disabled`) and all the related templates (`indexSuccess.php`, `error404Success.php`, `loginSuccess.php`, `secureSuccess.php`, `disabledSuccess.php`).
  * You can change the default module and action settings of the `settings.yml` file to use pages of your application.

Two other pages bear a symfony look and feel, and they also need to be customized before deployment to production. These pages are not in the `default` module, because they are called when symfony cannot run properly. Instead, you will find these default pages in the `sfConfig::get('sf_symfony_lib_dir')/exception/data/` directory:

  * `error.html.php`: Page called when an internal server error occurs in the production environment. In other environments (where debug is set to `true`), when an error occurs, symfony displays the full execution stack and an explicit error message (see Chapter 16 for details).
  * `unavailable.php`: Page called when a user requests a page while the application is disabled (with the `disable` task). It is also called while the cache is being cleared (that is, between a call to the `php symfony cache:clear` task and the end of this task execution). On systems with a very large cache, the cache-clearing process can take several seconds. Symfony cannot execute a request with a partially cleared cache, so requests received before the end of the process are redirected to this page.

To customize these pages, simply create `error/error.html.php` and `unavailable.php` in your project or application's `config/` directory. Symfony will use these templates instead of its own.

>**NOTE**
>To have requests redirected to the `unavailable.php` page when needed, you need to set the `check_lock` setting to `true` in the application `settings.yml`. The check is deactivated by default, because it adds a very slight overhead for every request.

### Optional Feature Activation

Some parameters of the `settings.yml` file control optional framework features that can be enabled or disabled. Deactivating unused features boosts performances a bit, so make sure to review the settings listed in Table 19-1 before deploying your application.

Table 19-1 - Optional Features Set Through `settings.yml`

Parameter               | Description | Default Value
----------------------- | ----------- | -------------
`use_database`          | Enables the database manager. Set it to `false` if you don't use a database. | `true`
`i18n`                  | Enables interface translation (see Chapter 13). Set it to `true` for multilingual applications. | `false`
`logging_enabled`       | Enables logging of symfony events. Set it to `false` when you want to turn symfony logging off completely. | `true`
`escaping_strategy`     | Enables the output escaping feature (see Chapter 7). Set it to `true` if you want data passed to your templates to be escaped. | `true`
`cache`                 | Enables template caching (see Chapter 12). Set it to `true` if one of your modules includes `cache.yml` file. The cache filter (`sfCacheFilter`) is enabled only if it is on. | `false` in development, `true` in production
`web_debug`             | Enables the web debug toolbar for easy debugging (see Chapter 16). Set it to `true` to display the toolbar on every page. | `true` in development, `false` in production
`check_symfony_version` | Enables the check of the symfony version for every request. Set it to on for automatic cache clearing after a framework upgrade. Leave it set to `false` if you always clear the cache after an upgrade. | `false`
`check_lock`            | Enables the application lock system, triggered by the `cache:clear` and `project:disable` tasks (see the previous section). Set it to `true` to have all requests to disabled applications redirected to the `sfConfig::get('sf_symfony_lib_dir')/exception/data/unavailable.php` page. | `false`
`compressed`            | Enables PHP response compression. Set it to `true` to compress the outgoing HTML via the PHP compression handler. | `false`

### Feature Configuration

Symfony uses some parameters of `settings.yml` to alter the behavior of built-in features such as form validation, cache, and third-party modules.

#### Output Escaping Settings

Output escaping settings control the way the variables are accessible in the template (see Chapter 7). The `settings.yml` file includes two settings for this feature:

  * The `escaping_strategy` setting can take the value `true`, or `false`.
  * The `escaping_method` setting can be set to `ESC_RAW`, `ESC_SPECIALCHARS`, `ESC_ENTITIES`, `ESC_JS`, or `ESC_JS_NO_ENTITIES`.

#### Routing Settings

The routing settings (see Chapter 9) are defined in `factories.yml`, under the `routing` key. Listing 19-1 show the default routing configuration.

Listing 19-1 - Routing Configuration Settings, in `frontend/config/factories.yml`

    routing:
      class: sfPatternRouting
      param:
        load_configuration: true
        suffix:             .
        default_module:     default
        default_action:     index
        variable_prefixes:  [':']
        segment_separators: ['/', '.']
        variable_regex:     '[\w\d_]+'
        debug:              %SF_DEBUG%
        logging:            %SF_LOGGING_ENABLED%
        cache:
          class: sfFileCache
          param:
            automatic_cleaning_factor: 0
            cache_dir:                 %SF_CONFIG_CACHE_DIR%/routing
            lifetime:                  31556926
            prefix:                    %SF_APP_DIR%

  * The `suffix` parameter sets the default suffix for generated URLs. The default value is a period (`.`), and it corresponds to no suffix. Set it to `.html`, for instance, to have all generated URLs look like static pages.
  * When a routing rule doesn't define the `module` or the `action` parameter, values from the `factories.yml` are used instead:
    * `default_module`: Default `module` request parameter. Defaults to the `default` module.
    * `default_action`: Default `action` request parameter. Defaults to the `index` action.
  * By default, route patterns identify named wildcards by a colon (`:`) prefix. But if you want to write your rules in a more PHP-friendly syntax, you can add the dollar (`$`) sign in the `variable_prefixes` array. That way, you can write a pattern like '/article/$year/$month/$day/$title' instead of '/article/:year/:month/:day/:title'.
  * The pattern routing will identify named wildcards between separators. The default separators are the slash and the dot, but you can add more if you want in the `segment_separators` parameter. For instance, if you add the dash (`-`), you can write a pattern like '/article/:year-:month-:day/:title'.
  * The pattern routing uses its own cache, in production mode, to speed up conversions between external URLs and internal URIs. By default, this cache uses the filesystem, but you can use any cache class, provided that you declare the class and its settings in the `cache` parameter. See Chapter 15 for the list of available cache storage classes. To deactivate the routing cache in production, set the `debug` parameter to `true`.

These are only the settings for the `sfPatternRouting` class. You can use another class for your application routing, either your own or one of symfony's routing factories (`sfNoRouting` and `sfPathInfoRouting`). With either of these two factories, all external URLs look like 'module/action?key1=param1'. No customization possible--but it's fast. The difference is that the first uses PHP's `GET`, and the second uses `PATH_INFO`. Use them mainly for backend interfaces.

There is one additional parameter related to routing, but this one is stored in `settings.yml`:

  * `no_script_name` enables the front controller name in generated URLs. The `no_script_name` setting can be on only for a single application in a project, unless you store the front controllers in various directories and alter the default URL rewriting rules. It is usually on for the production environment of your main application and off for the others.

#### Form Validation Settings

>**NOTE**
>The features described in this section are deprecated since symfony 1.1 and only work if you enable the `sfCompat10` plugin.

Form validation settings control the way error messages output by the `Validation` helpers look (see Chapter 10). These errors are included in `<div>` tags, and they use the `validation_error_ class` setting as a `class` attribute and the `validation_error_id_prefix` setting to build up the `id` attribute. The default values are `form_error` and `error_for_`, so the attributes output by a call to the `form_error()` helper for an input named `foobar` will be `class="form_error" id="error_for_foobar"`.

Two settings determine which characters precede and follow each error message: `validation_error_prefix` and `validation_error_suffix`. You can change them to customize all error messages at once.

#### Cache Settings

Cache settings are defined in `cache.yml` for the most part, except for two in `settings.yml`: `cache` enables the template cache mechanism, and `etag` enables ETag handling on the server side (see Chapter 15). You can also specify which storage to use for two all cache systems (the view cache, the routing cache, and the i18n cache) in `factories.yml`. Listing 19-2 show the default view cache factory configuration.

Listing 19-2 - View Cache Configuration Settings, in `frontend/config/factories.yml`

    view_cache:
      class: sfFileCache
      param:
        automatic_cleaning_factor: 0
        cache_dir:                 %SF_TEMPLATE_CACHE_DIR%
        lifetime:                  86400
        prefix:                    %SF_APP_DIR%/template

The `class` can be any of `sfFileCache`, `sfAPCCache`, `sfEAcceleratorCache`, `sfXCacheCache`, `sfMemcacheCache`, and `sfSQLiteCache`. It can also be your own custom class, provided it extends `sfCache` and provides the same generic methods for setting, retrieving and deleting a key in the cache. The factory parameters depend on the class you choose, but there are constants:

  * `lifetime` defines the number of seconds after which a cache part is removed
  * `prefix` is a prefix added to every cache key (use the environment in the prefix to use different cache depending on the environment). Use the same prefix for two applications if you want their cache to be shared.

Then, for each particular factory, you have to define the location of the cache storage.

 * for `sfFileCache`, the `cache_dir` parameter locates the absolute path to the cache directory
 * `sfAPCCache`, `sfEAcceleratorCache`, and `sfXCacheCache` don't take any location parameter, since they use PHP native functions for communicating with APC, EAccelerator or the XCache cache systems
 * for `sfMemcacheCache`, enter the hostname of the Memcached server in the `host` parameter, or an array of hosts in the `servers` parameter
 * for `sfSQLiteCache`, the absolute path to the SQLite database file should be entered in the `database` parameter

For additional parameters, check the API documentation of each cache class.

The view is not the only component to be able to use a cache. Both the `routing` and the `I18N` factories offer a `cache` parameter in which you can set any cache factory, just like the view cache. For instance, Listing 19-1 shows of the routing uses the file cache for its speedup tactics by default, but you can change it to whatever you want.

#### Logging Settings

Two logging settings (see Chapter 16) are stored in `settings.yml`:

  * `error_reporting` specifies which events are logged in the PHP logs. By default, it is set to `E_PARSE | E_COMPILE_ERROR | E_ERROR | E_CORE_ERROR | E_USER_ERROR` for the production environment (so the logged events are `E_PARSE`, `E_COMPILE_ERROR`, `E_ERROR`, `E_CORE_ERROR`, and `E_USER_ERROR`) and to `E_ALL | E_STRICT` for the development environment.
  * The `web_debug` setting activates the web debug toolbar. Set it to `true` only in the development and test environments.

#### Paths to Assets

The `settings.yml` file also stores paths to assets. If you want to use another version of the asset than the one bundled with symfony, you can change these path settings:

  * Files needed by the administration generator stored in `admin_web_dir`
  * Files needed by the web debug toolbar stored in `web_debug_web_dir`

#### Default Helpers

Default helpers, loaded for every template, are declared in the `standard_helpers` setting (see Chapter 7). By default, these are the `Partial`, `Cache`, and `Form` helper groups. If you use a helper group in all templates of an application, adding its name to the `standard_helpers` setting saves you the hassle of declaring it with `use_helper()` on each template.

#### Activated Modules

Activated modules from plug-ins or from the symfony core are declared in the `enabled_modules` parameter. Even if a plug-in bundles a module, users can't request this module unless it is declared in `enabled_modules`. The `default` module, which provides the default symfony pages (congratulations, page not found, and so on), is the only enabled module by default.

#### Character Set

The character set of the responses is a general setting of the application, because it is used by many components of the framework (templates, output escaper, helpers, and so on). Defined in the `charset` setting, its default (and advised) value is `utf-8`.


>**SIDEBAR**
>Adding Your application settings
>
>The `settings.yml` file defines symfony settings for an application. As discussed in Chapter 5, when you want to add new parameters, the best place to do so is in the `frontend/config/app.yml` file. This file is also environment-dependent, and the settings it defines are available through the sfConfig class with the `app_` prefix.
>
>
>     all:
>       creditcards:
>         fake:             false    # app_creditcards_fake
>         visa:             true     # app_creditcards_visa
>         americanexpress:  true     # app_creditcards_americanexpress
>
>
>You can also write an `app.yml` file in the project configuration directory, and this provides a way to define custom project settings. The configuration cascade also applies to this file, so the settings defined in the application `app.yml` file override the ones defined at the project level.

Extending the Autoloading Feature
---------------------------------

The autoloading feature, briefly explained in Chapter 2, exempts you from requiring classes in your code if they are located in specific directories. This means that you can just let the framework do the job for you, allowing it to load only the necessary classes at the appropriate time, and only when needed.

The `autoload.yml` file lists the paths in which autoloaded classes are stored. The first time this configuration file is processed, symfony parses all the directories referenced in the file. Each time a file ending with `.php` is found in one of these directories, the file path and the class names found in this file are added to an internal list of autoloading classes. This list is saved in the cache, in a file called `config/config_autoload.yml.php`. Then, at runtime, when a class is used, symfony looks in this list for the class path and includes the `.php` file automatically.

Autoloading works for all `.php` files containing classes and/or interfaces.

By default, classes stored in the following directories in your projects benefit from the autoloading automatically:

  * `myproject/lib/`
  * `myproject/lib/model`
  * `myproject/apps/frontend/lib/`
  * `myproject/apps/frontend/modules/mymodule/lib`

There is no `autoload.yml` file in the default application configuration directory. If you want to modify the framework settings--for instance, to autoload classes stored somewhere else in your file structure--create an empty autoload.yml file and override the settings of `sfConfig::get('sf_symfony_lib_dir')/config/config/autoload.yml` or add your own.

The autoload.yml file must start with an autoload: key and list the locations where symfony should look for classes. Each location requires a label; this gives you the ability to override symfony's entries. For each location, provide a `name` (it will appear as a comment in `config_autoload.yml.php`) and an absolute `path`. Then define if the search must be `recursive`, which directs symfony to look in all the subdirectories for `.php` files, and `exclude` the subdirectories you want. Listing 19-3 shows the locations used by default and the file syntax.

Listing 19-3 - Default Autoloading Configuration, in `sfConfig::get('sf_symfony_lib_dir')/config/config/autoload.yml`

    autoload:
      # plugins
      plugins_lib:
        name:           plugins lib
        path:           %SF_PLUGINS_DIR%/*/lib
        recursive:      true

      plugins_module_lib:
        name:           plugins module lib
        path:           %SF_PLUGINS_DIR%/*/modules/*/lib
        prefix:         2
        recursive:      true

      # project
      project:
        name:           project
        path:           %SF_LIB_DIR%
        recursive:      true
        exclude:        [model, symfony]

      project_model:
        name:           project model
        path:           %SF_LIB_DIR%/model
        recursive:      true

      # application
      application:
        name:           application
        path:           %SF_APP_LIB_DIR%
        recursive:      true

      modules:
        name:           module
        path:           %SF_APP_DIR%/modules/*/lib
        prefix:         1
        recursive:      true

A rule path can contain wildcards and use the file path parameters defined in the configuration classes (see the next section). If you use these parameters in the configuration file, they must appear in uppercase and begin and end with `%`.

Editing your own `autoload.yml` will add new locations to symfony's autoloading, but you may want to extend this mechanism and add your own autoloading handler to symfony's handler. As symfony uses the standard `spl_autoload_register()` function to manage class autoloading, you can register more callbacks in the application configuration class:

    [php]
    class frontendConfiguration extends sfApplicationConfiguration
    {
      public function initialize()
      {
        parent::initialize(); // load symfony autoloading first

        // insert your own autoloading callables here
        spl_autoload_register(array('myToolkit', 'autoload'));
      }
    }

When the PHP autoloading system encounters a new class, it will first try the symfony autoloading method (and use the locations defined in `autoload.yml`). If it doesn't find a class definition, all the other callables registered with `spl_autoload_register()` will be called, until the class is found. So you can add as many autoloading mechanisms as you want--for instance, to provide a bridge to other framework components (see Chapter 17).

Custom File Structure
---------------------

Each time the framework uses a path to look for something (from core classes to templates, plug-ins, configurations, and so on), it uses a path variable instead of an actual path. By changing these variables, you can completely alter the directory structure of a symfony project, and adapt to the file organization requirements of any client.

>**CAUTION**
>Customizing the directory structure of a symfony project is possible but not necessarily a good idea. One of the strengths of a framework like symfony is that any web developer can look at a project built with it and feel at home, because of the respect for conventions. Make sure you consider this issue before deciding to use your own directory structure.

### The Basic File Structure

The path variables are defined in the `sfProjectConfiguration` and `sfApplicationConfiguration` classes and stored in the `sfConfig` object. Listing 19-4 shows a listing of the path variables and the directory they reference.

Listing 19-4 - Default File Structure Variables, defined in `sfProjectConfiguration` and `sfApplicationConfiguration`

    sf_root_dir           # myproject/
    sf_apps_dir           #   apps/
    sf_app_dir            #     frontend/
    sf_app_config_dir     #       config/
    sf_app_i18n_dir       #       i18n/
    sf_app_lib_dir        #       lib/
    sf_app_module_dir     #       modules/
    sf_app_template_dir   #       templates/
    sf_cache_dir          #   cache/
    sf_app_base_cache_dir #     frontend/
    sf_app_cache_dir      #       prod/
    sf_template_cache_dir #         templates/
    sf_i18n_cache_dir     #         i18n/
    sf_config_cache_dir   #         config/
    sf_test_cache_dir     #         test/
    sf_module_cache_dir   #         modules/
    sf_config_dir         #   config/
    sf_data_dir           #   data/
    sf_lib_dir            #   lib/
    sf_log_dir            #   log/
    sf_test_dir           #   test/
    sf_plugins_dir        #   plugins/
    sf_web_dir            #   web/
    sf_upload_dir         #     uploads/

Every path to a key directory is determined by a parameter ending with `_dir`. Always use the path variables instead of real (relative or absolute) file paths, so that you will be able to change them later, if necessary. For instance, when you want to move a file to the `uploads/` directory in an application, you should use `sfConfig::get('sf_upload_dir')` for the path instead of `sfConfig::get('sf_root_dir').'/web/uploads/'`.

### Customizing the File Structure

You will probably need to modify the default project file structure if you develop an application for a client who already has a defined directory structure and who is not willing to change it to comply with the symfony logic. By overriding the `sf_XXX_dir` variables with `sfConfig`, you can make symfony work for a totally different directory structure than the default structure. The best place to do this is in the application `ProjectConfiguration` class for project directories, or `XXXConfiguration` class for applications directories.

For instance, if you want all applications to share a common directory for the template layouts, add this line to the `setup()` method of the `ProjectConfiguration` class to override the `sf_app_template_dir` settings:

    [php]
    sfConfig::set('sf_app_template_dir', sfConfig::get('sf_root_dir').DIRECTORY_SEPARATOR.'templates');

>**Note**
>Even if you can change your project directory structure by calling `sfConfig::set()`, it's better to use the dedicated methods defined by the project and application configuration classes if possible as they take care of changing all the related paths. For example, the `setCacheDir()` method changes the following constants: `sf_cache_dir`, `sf_app_base_cache_dir`, `sf_app_cache_dir`, `sf_template_cache_dir`, `sf_i18n_cache_dir`, `sf_config_cache_dir`, `sf_test_cache_dir`, and `sf_module_cache_dir`.

### Modifying the Project Web Root

All the paths built in the configuration classes rely on the project root directory, which is determined by the `ProjectConfiguration` file included in the front controller. Usually, the root directory is one level above the `web/` directory, but you can use a different structure. Suppose that your main directory structure is made of two directories, one public and one private, as shown in Listing 19-5. This typically happens when hosting a project on a shared host.

Listing 19-5 - Example of Custom Directory Structure for a Shared Host

    symfony/    # Private area
      apps/
      config/
      ...
    www/        # Public area
      images/
      css/
      js/
      index.php

In this case, the root directory is the `symfony/` directory. So the `index.php` front controller simply needs to include the `config/ProjectConfiguration.class.php` file as follows for the application to work:

    [php]
    require_once(dirname(__FILE__).'/../symfony/config/ProjectConfiguration.class.php');

In addition, use the `setWebDir()` method to change the public area from the usual `web/` to `www/`, as follows:

    [php]
    class ProjectConfiguration extends sfProjectConfiguration
    {
      public function setup()
      {
        // ...

        $this->setWebDir($this->getRootDir().'/../www');
      }
    }

Understanding Configuration Handlers
------------------------------------

Each configuration file has a handler. The job of configuration handlers is to manage the configuration cascade, and to do the translation between the configuration files and the optimized PHP code executable at runtime.

### Default Configuration Handlers

The default handler configuration is stored in `sfConfig::get('sf_symfony_lib_dir')/config/config/config_handlers.yml`. This file links the handlers to the configuration files according to a file path. Listing 19-6 shows an extract of this file.

Listing 19-6 - Extract of `sfConfig::get('sf_symfony_lib_dir')/config/config/config_handlers.yml`

    config/settings.yml:
      class:    sfDefineEnvironmentConfigHandler
      param:
        prefix: sf_

    config/app.yml:
      class:    sfDefineEnvironmentConfigHandler
      param:
        prefix: app_

    config/filters.yml:
      class:    sfFilterConfigHandler

    modules/*/config/module.yml:
      class:    sfDefineEnvironmentConfigHandler
      param:
        prefix: mod_
        module: yes

For each configuration file (`config_handlers.yml` identifies each file by a file path with wildcards), the handler class is specified under the `class` key.

The settings of configuration files handled by `sfDefineEnvironmentConfigHandler` can be made available directly in the code via the `sfConfig` class, and the param key contains a prefix value.

You can add or modify the handlers used to process each configuration file--for instance, to use INI or XML files instead of YAML files.

>**NOTE**
>The configuration handler for the `config_handlers.yml` file is `sfRootConfigHandler` and, obviously, it cannot be changed.

If you ever need to modify the way the configuration is parsed, create an empty `config_handlers.yml` file in your application's `config/` folder and override the `class` lines with the classes you wrote.

### Adding Your Own Handler

Using a handler to deal with a configuration file provides two important benefits:

  * The configuration file is transformed into executable PHP code, and this code is stored in the cache. This means that the configuration is parsed only once in production, and the performance is optimal.
  * The configuration file can be defined at different levels (project and application) and the final parameter values will result from a cascade. So you can define parameters at a project level and override them on a per-application basis.

If you feel like writing your own configuration handler, follow the example of the structure used by the framework in the `sfConfig::get('sf_symfony_lib_dir')/config/` directory.

Let's suppose that your application contains a `myMapAPI` class, which provides an interface to a third-party web service delivering maps. This class needs to be initialized with a URL and a user name, as shown in Listing 19-7.

Listing 19-7 - Example of Initialization of the `myMapAPI` Class

    [php]
    $mapApi = new myMapAPI();
    $mapApi->setUrl($url);
    $mapApi->setUser($user);

You may want to store these two parameters in a custom configuration file called `map.yml`, located in the application config/ directory. This configuration file might contain the following:

    api:
      url:  map.api.example.com
      user: foobar

In order to transform these settings into code equivalent to Listing 19-7, you must build a configuration handler. Each configuration handler must extend `sfConfigHandler` and provide an `execute()` method, which expects an array of file paths to configuration files as a parameter, and must return data to be written in a cache file. Handlers for YAML files should extend the `sfYamlConfigHandler` class, which provides additional facilities for YAML parsing. For the `map.yml` file, a typical configuration handler could be written as shown in Listing 19-8.

Listing 19-8 - A Custom Configuration Handler, in `frontend/lib/myMapConfigHandler.class.php`

    [php]
    <?php

    class myMapConfigHandler extends sfYamlConfigHandler
    {
      public function execute($configFiles)
      {
        // Parse the yaml
        $config = $this->parseYamls($configFiles);

        $data  = "<?php\n";
        $data .= "\$mapApi = new myMapAPI();\n";

        if (isset($config['api']['url'])
        {
          $data .= sprintf("\$mapApi->setUrl('%s');\n", $config['api']['url']);
        }

        if (isset($config['api']['user'])
        {
          $data .= sprintf("\$mapApi->setUser('%s');\n", $config['api']['user']);
        }

        return $data;
      }
    }

The `$configFiles` array that symfony passes to the `execute()` method will contain a path to all the `map.yml` files found in the `config/` folders. The `parseYamls()` method will handle the configuration cascade.

In order to associate this new handler with the `map.yml` file, you must create a `config_handlers.yml` configuration file with the following content:

    config/map.yml:
      class: myMapConfigHandler

>**NOTE**
>The `class` must either be autoloaded (that's the case here) or defined in the file whose path is written in a `file` parameter under the `param` key.

As with many other symfony configuration files, you can also register a configuration handler directly in your PHP code:

    sfContext::getInstance()->getConfigCache()->registerConfigHandler('config/map.yml', 'myMapConfigHandler', array());

When you need the code based on the `map.yml` file and generated by the `myMapConfigHandler` handler in your application, call the following line:

    [php]
    include(sfContext::getInstance()->getConfigCache()->checkConfig('config/map.yml'));

When calling the `checkConfig()` method, symfony looks for existing `map.yml` files in the configuration directories and processes them with the handler specified in the `config_handlers.yml` file, if a `map.yml.php` does not already exist in the cache or if the `map.yml` file is more recent than the cache.

>**TIP**
>If you want to handle environments in a YAML configuration file, the handler can extend the `sfDefineEnvironmentConfigHandler` class instead of `sfYamlConfigHandler`. Instead of calling the `parseYaml()` method to retrieve the configuration, you should call the `getConfiguration()` method: `$config = $this->getConfiguration($configFiles)`.

-

>**SIDEBAR**
>Using Existing configuration handlers
>
>If you just need to allow users to retrieve values from the code via `sfConfig`, you can use the `sfDefineEnvironmentConfigHandler` configuration handler class. For instance, to have the `url` and `user` parameters available as `sfConfig::get('map_url')` and `sfConfig::get('map_user')`, define your handler as follows:
>
>     config/map.yml:
>       class: sfDefineEnvironmentConfigHandler
>       param:
>         prefix: map_
>
>Be careful not to choose a prefix already used by another handler. Existing prefixes are `sf_`, `app_`, and `mod_`.

Summary
-------

The configuration files can heavily modify the way the framework works. Because symfony relies on configuration even for its core features and file loading, it can adapt to many more environments than just the standard dedicated host. This great configurability is one of the main strengths of symfony. Even if it sometimes frightens newcomers, who see in configuration files a lot of conventions to learn, it allows symfony applications to be compatible with a very large number of platforms and environments. Once you become a master of symfony's configuration, no server will ever refuse to run your applications!
