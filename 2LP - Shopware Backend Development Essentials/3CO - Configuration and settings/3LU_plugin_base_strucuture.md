---
title: "Plugin base structure"
slug: "plugin-base-structure"
description: "Learning unit about basic plugin structure"
icon: ""
authors: ["Micha Hobert"]
visibility: "public"
---

# Learning Objectives

By the end of this unit, you will be able to:

- Know where to place your plugin code
- Generate a basic plugin structure via the Shopware CLI
- Understand the main directories of a Shopware plugin

# Introduction

In this learning unit, you will learn about the basic structure of a Shopware plugin. Understanding the plugin structure is essential for developing custom plugins and extensions for Shopware.

# Plugin structure

Shopware plugins follow a specific directory structure that is important to understand for development. The structure is similar to Symfony bundles, as Shopware is built on the Symfony framework.

Here's a brief overview of the main directories:

```text
<plugin root>
├── src
│   └── MyPlugin
│       └── MyPlugin.php
├── Resources
│   ├── config
│   │   └── services.xml
│   ├── views
│   │   └── storefront
│   │       └── page
│   │           └── product-detail
│   │               └── my_plugin.html.twig
│   └── snippet
│       └── en_GB
│           └── storefront
│               └── my_plugin.en-GB.json
├── composer.json
└── README.md
```

This structure is a basic example of a Shopware plugin. The `src` directory contains the plugin's PHP code, while the `Resources` directory contains configuration files, views, and translations.

Don't worry if you don't understand everything yet. We will go through each part in detail in the following learning units. And good news on the creation of all those files and directories, Shopware CLI will help you with that.

# Plugin create command
Shopware comes with a powerful CLI that also helps you with the creation of plugins. To create a new plugin, run the following command:

```shell
bin/console plugin:create
```

This command will interactively ask you for the plugin name, namespace, and what kind of code should be generated. After answering the questions, the CLI will generate the plugin structure for you.


```shell
 Please enter a plugin name (PascalCase):
 > MyPlugin

 Please enter a plugin namespace (PascalCase):
 > MyPlugin

 Do you want to create an example console command? (yes/no) [yes]:
 > yes

 Do you want to create an example scheduled task? (yes/no) [yes]:
 > yes

 Do you want to create an example event subscriber? (yes/no) [yes]:
 > yes

 Do you want to create an example storefront controller? (yes/no) [yes]:
 > yes

 Do you want to create an example store-api route? (yes/no) [yes]:
 > yes

 Do you want to create entities? (yes/no) [yes]:
 > yes

 Please provide a list of entities (PascalCase, comma separated):
 > yes

 Do you want to create an example javascript plugin? (yes/no) [yes]:
 > no

 Do you want to create an example admin module? (yes/no) [yes]:
 > yes

 Do you want to create an example custom fieldset? (yes/no) [yes]:
 > yes
```

Pretty handy for your first playground and to get a feeling for the structure of a plugin and the file naming conventions.

![plugin-command](plugin-command.png)

## Plugin class

The MyPlugin class in the `src` directory is the main class of the plugin. It is responsible for registering the plugin and its services with Shopware. The class name should match the plugin name and namespace.

In it, you can define what should happen on installation, uninstallation, activation and update of the plugin. As well as postUpdate and postInstall methods.

```php
<?php declare(strict_types=1);

namespace MyPlugin;

use Shopware\Core\Framework\Plugin;
use Shopware\Core\Framework\Plugin\Context\ActivateContext;
use Shopware\Core\Framework\Plugin\Context\DeactivateContext;
use Shopware\Core\Framework\Plugin\Context\InstallContext;
use Shopware\Core\Framework\Plugin\Context\UninstallContext;
use Shopware\Core\Framework\Plugin\Context\UpdateContext;
use MihoTest\Service\CustomFieldsInstaller;

class MyPlugin extends Plugin
{
    public function install(InstallContext $installContext): void
    {
        // Do stuff such as creating a new payment method

        $this->getCustomFieldsInstaller()->install($installContext->getContext());
    }

    public function uninstall(UninstallContext $uninstallContext): void
    {
        parent::uninstall($uninstallContext);

        if ($uninstallContext->keepUserData()) {
            return;
        }

        // Remove or deactivate the data created by the plugin
    }

    public function activate(ActivateContext $activateContext): void
    {
        // Activate entities, such as a new payment method
        // Or create new entities here, because now your plugin is installed and active for sure

        $this->getCustomFieldsInstaller()->addRelations($activateContext->getContext());
    }

    public function deactivate(DeactivateContext $deactivateContext): void
    {
        // Deactivate entities, such as a new payment method
        // Or remove previously created entities
    }

    public function update(UpdateContext $updateContext): void
    {
        // Update necessary stuff, mostly non-database related
    }

    public function postInstall(InstallContext $installContext): void
    {
    }

    public function postUpdate(UpdateContext $updateContext): void
    {
    }

    private function getCustomFieldsInstaller(): CustomFieldsInstaller
    {
        if ($this->container->has(CustomFieldsInstaller::class)) {
            return $this->container->get(CustomFieldsInstaller::class);
        }

        return new CustomFieldsInstaller(
            $this->container->get('custom_field_set.repository'),
            $this->container->get('custom_field_set_relation.repository')
        );
    }
}
```

You did it, the plugin is now available in the administration panel and can be installed and activated. In the next learning unit, we will go through the possible plugin configurations.

If you are already familiar with the CLI you can also install and activate the plugin with it.

```shell
bin/console plugin:install --activate MyPlugin
```

## View and translation files

The `Resources/views` directory contains the plugin's view files. They contain all the `.twig` files that are used to render the plugin's output in the storefront. 

:::info
Twig is a versatile template engine for PHP. It is used in Shopware to render HTML output.
:::

Official documentation: [Twig](https://twig.symfony.com/)

The `Resources/snippet` directory contains translation files for the plugin. Adding those files will allow you to translate the plugin into different languages and can open up your plugin to a broader audience.


## Config folder and service.xml    
The service.xml file in the `Resources/config` directory contains the plugin's service definitions. It is used to define services that are used by the plugin.

For example, you can listen to events and react to them. Or you can define services like adding a custom controller to the storefront.

```xml
<?xml version="1.0" ?>

<container xmlns="http://symfony.com/schema/dic/services"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd">

    <services>
        <service id="Swag\MyPlugin\Service\AddDataToPage">
            <tag name="kernel.event_subscriber"/>
        </service>

        <service id="Swag\MyPlugin\Storefront\Controller\ExampleController" public="true">
            <call method="setContainer">
                <argument type="service" id="service_container"/>
            </call>
            <call method="setTwig">
                <argument type="service" id="twig"/>
            </call>
        </service>

    </services>

</container>
```

## Composer.json file
The composer.json file in the plugin root directory contains metadata about the plugin, such as its name, version, and dependencies. It is used by Composer to manage the plugin's dependencies and autoloading.

```json
{
  "name": "MyPlugin",
  "type": "shopware-platform-plugin",
  "description": "My first Shopware plugin",
  "version": "1.0.0",
  "license": "MIT",
  "require": {
    "shopware/core": "~6.6.0"
  },
  "extra": {
    "shopware-plugin-class": "MyPlugin",
    "label": {
      "de-DE": "Skeleton plugin",
      "en-GB": "Skeleton plugin"
    }
  },
  "autoload": {
    "psr-4": {
      "MyPlugin\\": "src/"
    }
  }
}
```


## Plugin commands

:::info
Getting all commands for the plugin CLI is easy, just run `bin/console plugin` and you will get a list of all available commands.
:::

```shell
Available commands for the "plugin" namespace:
  plugin:activate    Activate a plugin
  plugin:create      Creates a new plugin
  plugin:deactivate  Deactivates a plugin
  plugin:install     Installs a plugin
  plugin:list        Lists all plugins
  plugin:refresh     Refreshes the plugin list
  plugin:uninstall   Uninstall a plugin
  plugin:update      Updates a plugin
  plugin:update:all  Install all available plugin updates
  plugin:zip-import  Imports a plugin from a zip file
```


## Quiz

1. Can I install a plugin without the administration panel?
  [x] Yes
  [] No
2. What is the purpose of the `Resources/views` directory?
  [] To store translation files
  [x] To store view files
  [] To store configuration files
3. What is the purpose of the `Resources/config/services.xml` file?
  [] To configure packages used by the plugin
  [x] To define services used by the plugin
  [] There is no such file
4. What is the purpose of the `composer.json` file in a plugin?
  [x] To define the plugin's dependencies
  [] To define the plugin's services
  [] To define the plugin's views
  [x] Add metadata to the plugin
