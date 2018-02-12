---
title: Bootstrapping and Configuration
sidebar: basic_sidebar
permalink: basics_bootstrapping.html
folder: basics
---

## Programmatic configuration vs. configuration files
kiwi supports both, programmatic configurations (through configurators) and configuration files (through arrays) but
uses them for different use cases. 
Configuration files are used for environment (local, staging, production etc.) specific configurations (like database credentials). 
For environment unaware configuration we are using so called `configurators`. By default you can access the configurators during the
bootstrap process inside the `bootstrap` directory. Basically configurators are small helpers which allows you to configure through
setters etc. the corresponding component.

### Configuration files
All available configuration files will be merged during the bootstrap process.

{% include note.html content="Configurations inside `config/local` will overwrite any other configuration. By default the content
of this file is added to `.gitignore`. Its meant for environment specific configuration for the current machine." %} 

Configuration files must be named by the following pattern: `<name>.config.php` and must return an array.

## Bootstrap Process
The bootstrap process can be divided into 3 steps: providing configurator, initializing config services based on configurators 
and initializing the Servicemanager. In case of using a pre-compiled cache (should be used in production) only the last step
is executed. 


### Providing Configurators
All available configurators can be access by default inside the `bootstrap` directory. Every configurator will be injected
into his corresponding file. By design, configurators aren't immutable. 

### Initializing config services
Through the information collected by the configurator a config service can be created. The config service is immutable and can't 
be modified. Because these config services can be pre-compiled they must be serializable. 

### Initializing the Servicemanager
After all config services are created the servicemanager cann be initialized.
