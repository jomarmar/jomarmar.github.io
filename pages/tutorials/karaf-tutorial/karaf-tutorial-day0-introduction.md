---
permalink: /tutorials/karaf-tutorial/day0/
layout: page
categories: "tutorial"
tags: "karaf osgi java"
title: Karaf Tutorial Day 0
---

The aim of these tutorials is to provide a sample of a fully [Apache Karaf Application](https://karaf.apache.org), from the very early stages of design to the final production setup.
In **Day 0** tutorial we'll see: what is Karaf, installation and basic commands and deploy a  HelloWorld application.

{{ more }}

### Table of Contents
{:.no_toc}

* Table of Contents Placeholder
{:toc}

## What is Karaf?

[Apache Karaf Application](https://karaf.apache.org) is light OSGi container based on [Felix](http://felix.apache.org/)  or [Equinox](http://eclipse.org/equinox/). The difference between Karaf and other OSGi containers is the management features it brings with it.

Some of the outstanding features that I will use in this Tutorial series are:

- *Hot deployment and Dynamic configuration*
- *Provisioning*
- *Extensible BASH-like console*
- *Security Framework based on JAAS*
- *Web applications support*

After three or four years using Karaf one of the most astonishing features I've found is how easy a Karaf application can be integrated with other  external applications. With just a little bit of configuration and coding you can provide your application with access through: SSH commands, Web interface (WebServices, RESTful APIs).

## Installation and Startup

This tutorial will use Karaf 3.0.2 version. There are other Karaf branches such as Karaf 4.0.x (preview release, not for production use yet), Karaf 2.4.x (uses latest libraries for Aries and Pax, plus many improvements over 2.3.x) and Karaf 2.3.x. The reason why I use version 3.0.x in this tutorial is just that it is the current version I use in my latest projects: It is stable and it has many improvements over 2.3.x (see [here](http://www.liquid-reality.de/display/liquid/2013/12/28/10+reasons+to+switch+to+Apache+Karaf+3) for more information)

You can download the standard Karaf distribution from [here](http://karaf.apache.org/index/community/download.html).

### Apache Karaf Installation

In order to install Apache Karaf it is just enough to decompress the *.tar.gz or *.zip file to any folder in your machine. From now on this folder will be known as *KARAF_HOME*.

> **Note:** In order to run Karaf you need Java SDK installed in your machine. The minimum version recommended for Karaf 3.0.2 is JDK 6 but it is adviceable to use JDK 7 or higher.
>
> Also it is recommended to properly set the *JAVA_HOME* environment variable pointing to your JDK installation.

> **Note:** Karaf can be run on Windows / Linux OS



### Running Karaf: Basic commands

In order to start Karaf for the first time it is enough to execute the *karaf.(bat)* command. Once executed you should see the welcome screen:

{% highlight Bash shell scripts %}
[jmartinez@jfedora apache-karaf-3.0.2]$ ./bin/karaf
        __ __                  ____
       / //_/____ __________ _/ __/
      / ,<  / __ `/ ___/ __ `/ /_
     / /| |/ /_/ / /  / /_/ / __/
    /_/ |_|\__,_/_/   \__,_/_/

  Apache Karaf (3.0.2)

Hit '<tab>' for a list of available commands
and '[cmd] --help' for help on a specific command.
Hit '<ctrl-d>' or type 'system:shutdown' or 'logout' to shutdown Karaf.

karaf@root()>
{% endhighlight %}

This is the Karaf console. From here you can perform a lot of different operations such as: installing/starting/stopping bundles and features, configure services properties, manage JAAS implementation etc...

Commands are grouped by functionality, thus operations related with bundle management are grouped under *bundle* group, operations related with features handling under *feature* and so on...

To get help on a specific command is just enough to write:
{% highlight bash %}
<cmd> --help
{% endhighlight %}
The next list shows the commands I use more often:

- **info** *(shell:info)*: Prints system information.
- **list** *(bundle:list)*: Lists all installed bundles.
- **feature:list**: Lists all existing features available from the defined repositories. Option *-i* to see only installed features.
- **log:tail**: Continuously display log entries. Use ctrl-c to quit this command.
- **log:set**: To change the log level.
- **bundle:diag**: Shows diagnostic information why a bundle is not Active.

Besides there are a lot more commands we will be using as we developed our applications related with the installation/un-installation of bundles and features.

> For more information visit [http://karaf.apache.org](http://karaf.apache.org/manual/latest/index.html)
