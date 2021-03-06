---
layout: post
title:  Deploy with Command Line
categories: XAP97
parent: administrators-guide.html
weight: 500
---

{%summary%} {%endsummary%}

# Overview

GigaSpaces provides an interactive command line tool as part of the product. This can be started using gs.sh/bat command (referred to as **GigaSpaces CLI**).

This tool provides many commands that can be used to manage and gather information about the various GigaSpaces runtime components. This section describes the commands supported by GigaSpaces CLI.

# application

#### Syntax

    gs> deploy-application [-user xxx -password yyy] [-secured true/false] application_directory_or_zipfile

#### Description

Deploys an [application](./deploying-onto-the-service-grid.html#Application Deployment and Processing Unit Dependencies), which deploys one or more processing units in dependency order onto the service grid.


#### Options

{: .table .table-bordered}
|Option|Description|Value Format|
|:-----|:----------|:-----------|
| `-timeout` | Allows you to specify a timeout value (in milliseconds) when looking up the GSM to deploy to.{% wbr %}Defaults to `5000` milliseconds (5 seconds).| `-timeout [timeoutValue]`|
| `-deploy-timeout` | Timeout for deploy operation (in milliseconds),{% wbr %}otherwise blocks until all successful/failed deployment events arrive (default)" |`-deploy-timeout [timeoutValue]`|
| `-h` / `-help`  | Prints help | |
| `-secured` | Deploys a secured processing unit (implicit when using -user/-password) - [(CLI) Security](./command-line-interface-(cli)-security.html)| `-secured [true/false]`|
| `-user` `-password` | Deploys a secured processing unit propagated with the supplied user and password - [(CLI) Security](./command-line-interface-(cli)-security.html)| `-user xxx -password yyyy`|

{% togglecloak id=1 %}**Example**{% endtogglecloak %}
{% gcloak 1 %}


The following deploys the data-app example application (which includes a feeder and a processor).

    gs> deploy-application examples/data/dist.zip

The dist.zip file includes:

    application.xml
    feeder.jar
    processor.jar

application.xml file describes the application dependencies:

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
	xmlns:os-admin="http://www.openspaces.org/schema/admin"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
	                    http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.1.xsd
	                    http://www.openspaces.org/schema/admin http://www.openspaces.org/schema/{% currentversion %}/admin/openspaces-admin.xsd">

	<context:annotation-config />

	<os-admin:application name="data-app">

		<os-admin:pu processing-unit="processor.jar"
			cluster-schema="partitioned-sync2backup"
			number-of-instances="2" number-of-backups="1"
			max-instances-per-vm="1" max-instances-per-machine="0" />

		<os-admin:pu processing-unit="feeder.jar">
			<os-admin:depends-on name="processor" min-instances-per-partition="1"/>
		</os-admin:pu>

	</os-admin:application>
</beans>
{% endhighlight %}
{% endgcloak %}


# undeply application

#### Syntax

    gs> undeploy-application application_name

#### Description

Undeploys an [application](./deploying-onto-the-service-grid.html#Application Deployment and Processing Unit Dependencies) from the service grid, while respecting pu dependency order.



#### Options

{: .table .table-bordered}
|Option|Description|Value Format|
|:-----|:----------|:-----------|
| `-timeout` | Allows you to specify a timeout value (in milliseconds) when looking up the GSM to deploy to.{% wbr %}Defaults to `5000` milliseconds (5 seconds).| `-timeout [timeoutValue]`|
| `-undeploy-timeout` | Timeout for deploy operation (in milliseconds), otherwise blocks until all successful/failed deployment events arrive (default)" |`-undeploy-timeout [timeoutValue]`|
| `-h` / `-help`  | Prints help | |
| `-secured` | Deploys a secured processing unit (implicit when using -user/-password) - [(CLI) Security](./command-line-interface-(cli)-security.html)| `-secured [true/false]`|
| `-user` `-password` | Deploys a secured processing unit propagated with the supplied user and password - [(CLI) Security](./command-line-interface-(cli)-security.html)| `-user xxx -password yyyy`|

{% togglecloak id=2 %}**Example**{% endtogglecloak %}
{% gcloak 2 %}

The following undeploys the data-app example application (which includes a feeder and a processor).

    gs> undeploy-application data-app
{% endgcloak %}


# deploy PU

#### Syntax

    gs> deploy [processing unit jar file / directory location / name]

{% note %}
The `deploy` command replaces the `pudeploy` command and is identical to it in terms of supported arguments and options.
`pudeploy` is still supported but is considered a deprecated command and will be removed in future versions
{% endnote %}

#### Description

A Processing Unit can be easily deployed onto the Service Grid. In order to deploy a Processing Unit, the Processing Unit must follow the [processing unit directory structure](./the-processing-unit-structure-and-configuration.html).
Before deploying the processing unit you will need to jar it and then specify that jar file as the parameter to the `deploy` command. The deployment process will upload the jar file to all the GSMs it finds and unpack it under the `deploy` directory. It will then issue the deploy command.

{% tip %}
You may use the [GigaSpaces Universal Deployer](/sbp/universal-deployer.html) to deploy complex multi processing unit applications.
{% endtip %}

#### Third Party jars Location and Property Files

Third party jars should be placed within one of the following locations:

- Within the deployed jar under the lib folder - Good if you have relatively small amount of jars and few PU instances within the same GSC. These will be copied automatically by GigaSpaces during the deploy process to the `\gigaspaces-xap-premium\work` folder on all the machines running GSCs hosting the PU instances. You may control this folder location using the `com.gs.work` system property.
- Within the `\gigaspaces-xap-premium\lib\optional\pu-common` folder for each machine running GSCs - Each PU instance will have its own instance of the loaded class. Speed up the deployed time. You may control this folder location using the `com.gs.pu-common` system property.
- Within the `\gigaspaces-xap-premium\lib\platform\ext` folder for each machine running GSCs - All PUs class loaders will share the same loaded class. Speed up the deployed time. Optimize the JVM perm gem space usage since all 3rd party jars are loaded only once. You may control this folder location using the `com.gigaspaces.lib.platform.ext` system property.

Property files and other resources should be jared and placed within any of the above locations.

#### Deploy Command Options

{: .table .table-bordered}
|Option|Description|Value Format|
|:-----|:----------|:-----------|
| Processing Unit Location/Name -- **mandatory** | The location of the processing unit directory or jar file on your file system (see [this page](./deploying-onto-the-service-grid.html)).{% wbr %}If you are using a few options in the `deploy` command, pass this option as the **last parameter**.{% wbr %}For example: `gs> deploy hello-world.jar`{% wbr %}(`hello-world.jar` is the processing jar file). | |
| `-cluster` |Allows you to control the clustering characteristics of the processing unit.{% wbr %}The cluster option is a simplified option that overrides the cluster part of the processing unit's built in SLA (if such exists).{% wbr %}The following options are available (used automatically by any embedded space included in the Processing Unit):{% wbr %}- `schema` -- the cluster schema used by the Processing Unit.{% wbr %}- `total_members` -- the number of instances, optionally followed by the number of backups{% wbr %}(number of backups is required only if the `partitioned-sync2backup` schema is used). | `-cluster schema=[schema name]`{% wbr %}`total_members=`{% wbr %}`numberOfInstances[,numberOfBackups]` |
| `-properties` | Allows you to control [deployment properties](./deployment-properties.html). | `-properties [bean name] location` |
| `-sla` | Allows you to specify a link (default to file-system) to a Spring XML configuration, holding the SLA definition. | `-sla [slaLocation]` |
| `-zones` | Allows you to specify a list of deployment zones that are to restrict that the deployment to specific GSCs. | `-zones [zoneName1 zoneName2 ... ]` |
| `-timeout` | Allows you to specify a timeout value (in milliseconds) when looking up the GSM to deploy to.
  Defaults to `5000` milliseconds (5 seconds).| `-timeout [timeoutValue]`|
| `-override-name` | Allows you to specify an override name for the deployed Processing Unit{% wbr %}(a different name than the directory name under `deploy`).{% wbr %}Mainly used when using a Processing Unit as a template.| `-override-name [processing unit name]` |
| `-max-instances-per-vm` | Allows you to set the SLA number of instances per VM | |
| `-max-instances-per-machine` | Allows you to set the SLA number of instances per machine | |
| `-max-instances-per-zone` | Allows you to set the SLA number of instances per zone in the format of `zoneX/number,zoneY/number` | |
| `h` / `help`  | Prints help | |
| `-secured` | Deploys a secured processing unit (implicit when using -user/-password) - [(CLI) Security](./command-line-interface-(cli)-security.html)| `-secured [true/false]`|
| `-user` `-password` | Deploys a secured processing unit propagated with the supplied user and password - [(CLI) Security](./command-line-interface-(cli)-security.html)| `-user xxx -password yyyy`|

{% tip %}
You may use the [Primary-Backup Zone Controller](/sbp/primary-backup-zone-controller.html) to deploy primary and backup on specific different zones.
{% endtip %}

{% togglecloak id=3 %}**Example**{% endtogglecloak %}
{% gcloak 3 %}

The following deploys a processing unit jar file named `data-processor.jar` using the `sync_replicated` cluster schema with 2 instances (`total_members`).

    gs> deploy -cluster schema=sync_replicated total_members=2 data-processor.jar

The following deploys a processing unit archive called `data-processor.jar` using deployment properties file called `pu.properties`.

    gs> deploy -properties file://config/pu.properties data-processor.jar

The following deploys a processing unit archive called `data-processor.jar` using an SLA element read from an external `sla.xml` file.

    gs> deploy -sla file://config/sla.xml data-processor.jar

The following example deploys a `partitioned-sync2backup` space cluster with the name `mySpace` for both the processing unit and the Space it contains.

    deploy -cluster schema=partitioned-sync2backup total_members=2,1 -override-name mySpace -properties embed://dataGridName=mySpace myPUFolder

{% tip %}
Multiple deployment properties can be injected by having ; between each property - see below example:

{% highlight java %}
>gs deploy -cluster schema=partitioned-sync2backup total_members=10,1
-properties "embed://dataGridName=myIMDG;space-config.proxy.router.active-server-lookup-timeout=5000;space-config.engine.max_threads=256"
-override-name myPU /tmp/myPu.jar
{% endhighlight %}
{% endtip %}

{% endgcloak %}

# undeploy app

#### Syntax

    gs> undeploy-application application_name

#### Description

Undeploys an [application](./deploying-onto-the-service-grid.html#Application Deployment and Processing Unit Dependencies) from the service grid, while respecting pu dependency order.


#### Options

{: .table .table-bordered}
|Option|Description|Value Format|
|:-----|:----------|:-----------|
| `-timeout` | Allows you to specify a timeout value (in milliseconds) when looking up the GSM to deploy to.
  Defaults to `5000` milliseconds (5 seconds).| `-timeout [timeoutValue]`|
| `-undeploy-timeout` | Timeout for deploy operation (in milliseconds), otherwise blocks until all successful/failed deployment events arrive (default)" |`-undeploy-timeout [timeoutValue]`|
| `-h` / `-help`  | Prints help | |
| `-secured` | Deploys a secured processing unit (implicit when using -user/-password) - [(CLI) Security](./command-line-interface-(cli)-security.html)| `-secured [true/false]`|
| `-user` `-password` | Deploys a secured processing unit propagated with the supplied user and password - [(CLI) Security](./command-line-interface-(cli)-security.html)| `-user xxx -password yyyy`|

{% togglecloak id=4 %}**Example**{% endtogglecloak %}
{% gcloak 4 %}

The following undeploys the data-app example application (which includes a feeder and a processor).

    gs> undeploy-application data-app

{% endgcloak %}


# deploy memcached

#### Syntax

    gs> deploy-memcached [-sla ...] [-cluster ...] [-properties ...] [-user xxx -password yyy] [-secured true/false] space_url

#### Description

Deploys a [memcached-enabled space](./memcached-api.html), which exposes the [memcached protocol](http://code.google.com/p/memcached/wiki/NewProtocols), onto the service grid.

#### Options

{: .table .table-bordered}
|Option|Description|Value Format|
|:-----|:----------|:-----------|
| space_url | The url of the space, can be embedded, eg: `/./myMemcachedSpace`, or remote eg: `jini://*/*/myMemcachedSpace` | |
| `-cluster` |Allows you to control the clustering characteristics of the processing unit. {% wbr %}The cluster option is a simplified option that overrides the cluster part of the processing unit's built in SLA (if such exists). {% wbr %}The following options are available (used automatically by any embedded space included in the Processing Unit):{% wbr %}- `schema` -- the cluster schema used by the Processing Unit.{% wbr %}- `total_members` -- the number of instances, optionally followed by the number of backups {% wbr %}(number of backups is required only if the `partitioned-sync2backup` schema is used). | `-cluster schema=[schema name] total_members=numberOfInstances[,numberOfBackups]` |
| `-properties` | Allows you to control [deployment properties](./deployment-properties.html). | `-properties [bean name] location` |
| `-sla` | Allows you to specify a link (defaults to file-system) to a Spring XML configuration, holding the SLA definition. | `-sla [slaLocation]` |
| `-zones` | Allows you to specify a list of deployment zones that are to restrict that the deployment to specific GSCs. | `-zones [zoneName1, zoneName2 ... ]` |
| `-timeout` | Allows you to specify a timeout value (in milliseconds) when looking up the GSM to deploy to.{% wbr %}Defaults to `5000` milliseconds (5 seconds).| `-timeout [timeoutValue]`|
| `-max-instances-per-vm` | Allows you to set the SLA number of instances per VM | |
| `-max-instances-per-machine` | Allows you to set the SLA number of instances per machine | |
| `-max-instances-per-zone` | Allows you to set the SLA number of instances per zone in the format of `zoneX/number,zoneY/number` | |
| `h` / `help`  | Prints help | |
| `-secured` | Deploys a secured processing unit (implicit when using -user/-password) - [(CLI) Security](./command-line-interface-(cli)-security.html)| `-secured [true/false]`|
| `-user` `-password` | Deploys a secured processing unit propagated with the supplied user and password - [(CLI) Security](./command-line-interface-(cli)-security.html)| `-user xxx -password yyyy`|

{% tip %}
You can use the [GigaSpaces Universal Deployer](/sbp/universal-deployer.html) to deploy complex multi processing unit applications.
{% endtip %}

{% togglecloak id=5 %}**Example**{% endtogglecloak %}
{% gcloak 5 %}

The following deploys a memcached-enabled space named `mySpace` using the `partitioned-sync2backup` cluster schema with 2 primaries and 1 primary per backup.

    gs> deploy-memcached -cluster schema=partitioned-sync2backup total_members=2,1 mySpace

The following deploys a memcached-enabled space called `mySpace` using an SLA element read from an external `sla.xml` file.

    gs> deploy-space -sla file://config/sla.xml mySpace
{% endgcloak %}


# deploy space

#### Syntax

    gs> deploy-space [space name]

#### Description

A Space only Processing Unit can be easily deployed onto the Service Grid.

#### Options

{: .table .table-bordered}
|Option|Description|Value Format|
|:-----|:----------|:-----------|
| Space Name -- **mandatory** | The name of the space to be deployed.| |
| `-cluster` |Allows you to control the clustering characteristics of the space.{% wbr %}The following options are available (used automatically by any embedded space included in the Processing Unit):{% wbr %}- `schema` -- the cluster schema used by the Processing Unit.{% wbr %}- `total_members` -- the number of instances, optionally followed by the number of backups {% wbr %}  (number of backups is required only if the `partitioned-sync2backup` schema is used). | `-cluster schema=[schema name]`{% wbr %}`total_members=numberOfInstances[,numberOfBackups]` |
| `-properties` | Allows you to control [deployment properties](./deployment-properties.html). | `-properties [bean name] location` |
| `-sla` | Allows you to specify a link (default to file-system) to a Spring XML configuration, holding the SLA definition. | `-sla [slaLocation]` |
| `-zones` | Allows you to specify a list of deployment zones that are to restrict that the deployment to specific GSCs. | `-zones [zoneName1, zoneName2 ... ]` |
| `-max-instances-per-vm` | Allows you to set the SLA number of instances per VM | |
| `-max-instances-per-machine` | Allows you to set the SLA number of instances per machine | |
| `-max-instances-per-zone` | Allows you to set the SLA number of instances per zone in the format of `zoneX/number,zoneY/number` | |
| `h` / `help`  | Prints help | |
| `-secured` | Deploys a secured processing unit (implicit when using -user/-password) - [(CLI) Security](./command-line-interface-(cli)-security.html)| `-secured [true/false]`|
| `-user` `-password` | Deploys a secured processing unit propagated with the supplied user and password - [(CLI) Security](./command-line-interface-(cli)-security.html)| `-user xxx -password yyyy`|

{% tip %}
You may use the [GigaSpaces Universal Deployer](/sbp/universal-deployer.html) to deploy complex multi processing unit applications.
You may use the [Primary-Backup Zone Controller](/sbp/primary-backup-zone-controller.html) to deploy primary and backup instances on different zones.
{% endtip %}

{% togglecloak id=6 %}**Example**{% endtogglecloak %}
{% gcloak 6 %}

The following deploys a space named `mySpace` using the `sync_replicated` cluster schema with 2 instances (`total_members`).

    gs> deploy-space -cluster schema=sync_replicated total_members=2 mySpace

The following deploys a space named `mySpace` using deployment properties file called `pu.properties`.

    gs> deploy-space -properties file://config/pu.properties mySpace

The following deploys a space called `mySpace` using an SLA element read from an external `sla.xml` file.

    gs> deploy-space -sla file://config/sla.xml mySpace

{% endgcloak %}

# task

#### Syntax

    usage: task ant-file [target=target-name]

#### Description

The `task` command submits a task in the form of an Ant configuration file..

#### Options

{: .table .table-bordered}
| Option | Description |
|:-------|:------------|
| `ant-file` | The name of the Ant configuration file, an XML file representing the task. The file must reside in the current directory. |
| `list-of-machines` | A comma-separated list of hostnames or of IP addresses, or the name of a file containing such a list, saying where to submit the Ant configuration file. By default, if machines are available, you receive a list to choose from. If no machines are currently available, are prompted to start an HTTP server. |
