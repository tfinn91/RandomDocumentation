Running Graphite, Grafana, StatsD inside Docker
==============================================





##BUILD INSTRUCTIONS


###Install Graphite

Graphite is an excellent tool for organizing and rendering visual representations of data gathered from your system. It is highly flexible and can be configured so that you can gain the benefits of both detailed representation and broad overviews of the performance and health of the metrics you are tracking.

To get started, you need to download and install the Graphite components. This guide is written with an ubuntu 14.04 server in mind. Graphite is made of several components: the web application, a storage backend called Carbon, and the database library called whisper.

To begin the installation:

```bash
sudo apt-get update
sudo apt-get install graphite-web graphite-carbon
```

It will ask you whether you want Carbon to remove the database files if you ever decide to purge the installation. Select 'No' and hit enter so that you will not destroy your stats. If you need to start fresh, you can always manually remove the files (kept in `var/lib/graphite/whisper`).

####Configure Database for Django

Although the Graphite data itself is handled by Carbon and the whisper database library, the web application is a Django Python application, and needs to store its data somewhere.

You can install the database software and the helper packages with: 

```bash
sudo apt-get install postgresql libpq-dev python-psycopg2
```

This will install the database software, as well as the Python libraries that Graphite will use to connect to and communicate with the database.

After the database software is installed, you'll need to create a PostgreSQL user and database for Graphite to use. You can then create a database username and give it a password (change to a secure password). After that, create the database and give the previously created user control of it. You can then exit:

```bash
sudo -u postgres psql
CREATE USER graphite WITH PASSWORD 'password';
CREATE DATABASE graphite WITH OWNER graphite;
\q
```

You will receive a message saying "Postgres could not save the file history." This is not a problem for so you can just continue. 

####Configure Graphite Web-App

Open the Graphite web app configuration file (this guide will use the nano editor):

```bash
sudo nano /etc/graphite/local_settings.py
```

Now you need to specify the timezone. This will affect the time displayed on the graphs. Set it to your time zone as specified by the "TZ*" column in this [wikipedia list of time zones][1].

```python
TIME_ZONE = 'America/Denver'
```

Now, set the secret key that will be used as a salt when creating hashes. Uncomment the `SECRET_KEY` parameter and change the value to something unique:

```python
SECRET_KEY = 'a_secret_string'
```

You also want to configure authentication for saving graph data. When you sync the database, you'll be able to create a user account, but you need to enable authentication by uncommenting this line:

 ```python
 USE_REMOTE_USER_AUTHENTICATION = True
 ```


Scroll down until you see the `DATABASES` definition. Change the values to reflect your Postgres information. You should change the `NAME`, `ENGINE`, `USER`, `PASSWORD`, and `HOST` keys. It should look something like this (edit to reflect your changes):

```python
DATABASES = {
    'default': {
        'NAME': 'graphite',
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'USER': 'graphite',
        'PASSWORD': 'password',
        'HOST': '127.0.0.1',
        'PORT': ''
    }
}
```


Now you need to sync the database to create the correct structure. You can do this by typing:

```bash
sudo graphite-manage syncdb
```

You will be asked to create a superuser account for the database. Create a new user so that you can sign into the interface. You can call this whatever you want. This will allow you to save your graphs and modify the interface.

####Configure Carbon

Carbon is the Graphite storage backend. First, enable the carbon service to start at boot. Do this by opening the service configuration file:

```bash
sudo nano /etc/default/graphite-carbon
```

Change the following value to true:

```
CARBON_CACHE_ENABLED=true
```

Save and close the file. Next, open the Carbon configuration file:

```bash
sudo nano /etc/carbon/carbon.conf
```

Turn on log rotation by adjusting setting this directive to true:
```
ENABLE_LOGROTATION = True
```

Save and close the file. Now, open the storage schema file. This tells Carbon how long to store values and how detailed these values should be:

```bash
sudo nano /etc/carbon/storage-schemas.conf
```

The file currently has two sections defined. The first one is for deciding what to do with data coming from Carbon itself. Carbon is actually configured to store some metrics of its own performance. The bottom definition is a catch-all that is designed to apply to any data that hasn't been matched by another section. It defines a default policy.

The retention policy is defined by sets of numbers. Each set consists of a metric interval (how often a metric is recorded), followed by a colon and then the length of time to store those values. You can define multiple sets of numbers separated by commas.

To demonstrate, define a new schema that will match a test value that you'll use later on. Before the default section, add another section for your test values. Make it look like this:

```
[test]
pattern = ^test\.
retentions = 10s:10m,1m:1h,10m:1d
```

This will match any metrics beginning with "test.". It will store the data it collects three times, in varying detail. The first archive definition (10s:10m) will create a data point every ten seconds. It will store the values for only ten minutes.

The second archive (1m:1h) will create a data point every minute. It will gather all of the data from the past minute (six points, since the previous archive creates a point every ten seconds) and aggregate it to create the point. By default, it does this by averaging the points, but you can adjust this later. It stores the data at this level of detail for one hour.

The last archive that will be created (10m:1d) will make a data point every 10 minutes, aggregating the data in the same way as the second archive. It will store the data for one day.


**Note**: It is important to realize that if you send Graphite data points more frequently than the shortest archive interval length, some of your data will be lost!


Save and close the file. When you are finished, you can start Carbon by typing:

```bash
sudo service carbon-cache start
```

####Install and Configure Apache

Because Graphite includes configuration files for Apache, this installation guide will focus on installing using Apache. You can install the necessary components by typing:

```bash
sudo apt-get install apache2 libapache2-mod-wsgi
```

When the installation is complete, disable the default virtual host file, since it conflicts with the new file:

```bash
sudo a2dissite 000-default
```

Next, copy the Graphite Apache virtual host file into the available sites directory:

```bash
sudo cp /usr/share/graphite-web/apache2-graphite.conf /etc/apache2/sites-
available
```

Then enable the virtual host file aand reload the service to implement the changes by typing the following

```bash
sudo a2ensite apache2-graphite
sudo service apache2 reload
```

This is what it takes to get Graphite up and running. Open up a web browser and visit your server's domain name or ip address. The Graphite dashboard should appear! You will want to login so that you can save and share graphs. Login with the username and password configured when syncing the Django database.

For more information on how to use the Graphite dashboard and send test data, please see https://www.digitalocean.com/community/tutorials/how-to-install-and-use-graphite-on-an-ubuntu-14-04-server#CheckingouttheWebInterface. This gives a good tutorial on how to work with the data inside G
raphite. This site is from where the information on this wiki page was pulled. 


###Install CollectD

After Graphite is up and running, the task now is to send data to Graphite so that it can be analyzed and displayed. Collecd is a system statistics gatherer that can collect and organize metrics about your server and running services.

Refresh the local package index and then install by typing:

```bash
sudo apt-get update
sudo apt-get install collectd collectd-utils
```

This will install the daemon and a helper control interface. Now it just needs to be configured to work with Graphite.

####Configure CollectD

Begin by opening the collectd configuration file in your editor with root privileges:

```bash
sudo nano /etc/collectd/collectd.conf
```

The first thing that you should set is the hostname of the machine that you are on. Collectd **can be used to send information to a remote Graphite server**, but it is being used on the same machine for the sake of this guide. You can choose whatever name you'd like:

```
Hostname "graph_host"
```

Now you need to configure some plugins found in the .conf file. For this guide, the following plugins will need to be uncommented (and edited for some). So, uncomment the following plugins:

```
LoadPlugin apache
LoadPlugin cpu
LoadPlugin df
LoadPlugin entropy
LoadPlugin interface
LoadPlugin load
LoadPlugin memory
LoadPlugin processes
LoadPlugin rrdtool
LoadPlugin users
LoadPlugin write_graphite
```


Scroll down the page and you will see block definition for each plugin. 

**Apache Plugin**: The Apache plugin is necessary as Apache was installed to serve Graphite. Configure the Apache plugin with a simple section that looks like this (be sure to enter your domain name or IP in the URL section):

```
<Plugin apache>
    <Instance "Graphite">
        URL "http://DOMAIN NAME OR IP HERE/server-status?auto"
        Server "apache"
    </Instance>
</Plugin>
```

**df Plugin**: This tells us how full our disks are. Add a simple configuration that looks like this:

```
<Plugin df>
    Device "/dev/vda"
    MountPoint "/"
    FSType "ext3"
</Plugin>
```

`Device` should point to the device name of the drive on your system. In the terminal, simply type `df` and you will see something similar to the following:

```
Filesystem     1K-blocks    Used Available Use% Mounted on
/dev/vda       61796348 1766820  56867416   4% /
none                   4       0         4   0% /sys/fs/cgroup
udev             2013364      12   2013352   1% /dev
tmpfs             404836     340    404496   1% /run
```

Notice `/dev/vda`. Because it shows vda, make sure that in the `Device` section of the df plugin, the line reads `Device "/dev/vda"`

**Interface Plugin**: Choose the networking interface you wish to monitor (eth0 here by default):

```
<Plugin interface>
    Interface "eth0"
    IgnoreSelected false
</Plugin>
```


**Write_Graphite Plugin**: This will tell ColletD how to connect to the Graphite instance. Make the section look like the following:


```
<Plugin write_graphite>
    <Node "graphing">
        Host "localhost"
        Port "2003"
        Protocol "tcp"
        LogSendErrors true
        Prefix "collectd."
        StoreRates true
        AlwaysAppendDS false
        EscapeCharacter "_"
    </Node>
</Plugin>
```

This tells our daemon how to connect to Carbon in order to pass off its data. This specifies that it should look to the local computer on port 2003, which Carbon uses to listen for TCP connections.

Save and close the configuration file when finished. 

####Configure Apache for Reporting

In the configuration file, Apache stats tracking was enabled. However, you still need to configure Apache to allow this. In the Apache virtual hosts file that has been have enabled for Graphite, all you need to do is add a simple location block that will tell Apache to report stats.

Open the file in your text editor:

```bash
sudo nano /etc/apache2/sites-available/apache2-graphite.conf
```

You will see something similar to the following:

```
Alias /content/ /usr/share/graphite-web/static/
    <Location "/content/">
        SetHandler None
    </Location>

    ErrorLog ${APACHE_LOG_DIR}/graphite-web_error.log
```

After the `</Location>` tag and before the `ErrorLog` line, add the following code. This block of code ensures that Apache will serve statistics at the `/server-status` page. 

```
    <Location "/server-status">
        SetHandler server-status
        Require all granted
    </Location>
```

Save and close the file. Once finished, reload the Apache service:

```bash
sudo service apache2 reload
```

Check to make sure everything is working correctly by visiting the page in your web browser. Just go to your domain, followed by /server-status:

```
http://YOUR_DOMAIN_NAME_OR_IP/server-status
```

You should see a page with "Apache Server Status For `your_domain_name_or_IP`"

####Configure Carbon for CollectD's Data

Collectd has been configured to gather statistics about your services, now you need to adjust Graphite to handle the data it receives correctly. Start by creating a storage schema definition. Open up the storage schema configuration file:

```bash
sudo nano /etc/carbon/storage-schemas.conf
```

Add the following block **above** the `default` block or else it will never get applied:

```
[collectd]
pattern = ^collectd.*
retentions = 10s:1d,1m:7d,10m:1y
```

This block tells Graphite to store collectd information at intervals of ten seconds for one day, at one minute for seven days, and intervals of ten minutes for one year. Once finished you can save and close the file.

####Reload Services

Collectd has now been configured and Graphite knows how to handle the data received from Collectd. Now you just have to reload `carbon-cache` and `collectd`:

```bash
sudo service carbon-cache stop
sudo service collectd stop        ## Wait about 10 seconds here
sudo service carbon-cache start
sudo service collectd start
```

Now if you open up your web browser and refresh the Graphite web-application, you should be able to see collectd information on the tree on the left!

For more information on collectd please refer to [collectd's homepage][2].



###Install StatsD

StatsD is a lightweight statistics gathering daemon that can be used to collect arbitrary statistics. StatsD flushes stats to Graphite in sync with Graphite's configured write interval. To do this, it aggregates all of the data between flush intervals and creates single points for each statistic to send to Graphite.

In this way, StatsD lets applications work around the effective rate-limit for sending Graphite stats. It has many libraries written in different programming languages that make it trivial to build in stats tracking with your applications.

####node.js version

You need git so that in order to clone the repository. You also need node.js because StatsD is a node application. There are a few other packages are necessary that will allow you to build an Ubuntu package:
```bash
sudo apt-get install git nodejs devscripts debhelper
```

For the sake of this guide, create the package in the home directory. More specifically, create a directory called "build" in your home directory to complete this process:

```bash
mkdir ~/build
```

Now you need to clone StatsD into that directory:
```bash
cd ~/build
git clone https://github.com/etsy/statsd.git
```

Now you need to build and install it:

```bash
cd statsd
dpkg-buildpackage      ##This creates a .deb file in ~/build
cd ..
```
Before you install the package, you should stop the Carbon service. The reason for this is that the StatsD service will immediately start sending information when it is installed and it is not yet configured properly.

```bash
sudo service carbon-cache stop
```

Install the package:

```bash
sudo dpkg -i statsd*.deb
```

Now you can reload carbon and configure StatsD. To do this you need to stop StatsD and start carbon:

```bash
sudo service statsd stop
sudo service carbon-cache start
```

####Configure StatsD

The first thing that needs to happen is the modification of the config file. This can be done by opening the file:

```bash
sudo nano /etc/statsd/localConfig.js
```

The only thing that is necessary to do is to turn off what StatsD calls "Legacy Namespacing." StatsD uses this to organize its data in a different way. In more recent versions, however, it has standardized on a more intuitive structure. This just allows you to use more sensible naming conventions. Make the file look like the following:

```js
{
  graphitePort: 2003
, graphiteHost: "localhost"
, port: 8125
, graphite: {
    legacyNamespace: false
  }
}
```

Save and close the file once finished. 

####Storage Schemas

Similar to what you did for collectd, it is necessary to define some more storage-schemas, in this case for StatsD. Open the storage-schema file:

```bash
sudo nano /etc/carbon/storage-schemas.conf
```

The retention policy that was created for collectd is the exact same that will be used in this case for StatsD. Add the following anywhere above the `default` block. If it is below the `default` block, it will never be applied!!

```
[statsd]
pattern = ^stats.*
retentions = 10s:1d,1m:7d,10m:1y
```

Save and close the file once the above `statsd` block has been added.

####Data Aggregation Configuration

StatsD sends data in a very specific way, so you can easily ensure that you are aggregating the data correctly by matching the correct patterns. Open the file in your editor:

```bash
sudo nano /etc/carbon/storage-aggregation.conf
```

The following is taken from Etsy's Github page where it talks about setting up graphite for StatsD. The file should look like the following:

```
[min]
pattern = \.min$
xFilesFactor = 0.1
aggregationMethod = min

[max]
pattern = \.max$
xFilesFactor = 0.1
aggregationMethod = max

[count]
pattern = \.count$
xFilesFactor = 0
aggregationMethod = sum

[lower]
pattern = \.lower(_\d+)?$
xFilesFactor = 0.1
aggregationMethod = min

[upper]
pattern = \.upper(_\d+)?$
xFilesFactor = 0.1
aggregationMethod = max

[sum]
pattern = \.sum$
xFilesFactor = 0
aggregationMethod = sum

[gauges]
pattern = ^.*\.gauges\..*
xFilesFactor = 0
aggregationMethod = last

[default_average]
pattern = .*
xFilesFactor = 0.5
aggregationMethod = average
```

Save and close the file once it looks similar to above.

Now you just want to restart all of the services:

```bash
sudo service carbon-cache stop
sudo service carbon-cache start
sudo service statsd start
```

Okay once that is finished, refresh the graphite web-application and StatsD should appear in the tree on the left.





####bitly/statsdaemon version

This is the Go port of [Etsy's Statsd][3], written in Go (originally based on amir/gographite). Installation is very easy as well.

Now you are ready to get and install the package

```bash
go get github.com/bitly/statsdaemon
```

A very useful piece of documentation is provided by GoDoc. The document covers [package statsd][4] and how to create applications that send metrics to StatsD.
This document also goes through examples on how to connect to a StatsD client through your applications. Read it!!


###Install Grafana

Grafana is the graphite front-end that will be used. It is fairly easy to install and get set up as well. Besides Graphite, it only has one optional external dependency and that is Elasticsearch. Elasticsearch is used to store, load and search for dashboards. You can use Grafana withoug Elasticsearch but for the sake of this guide, Elasticsearch will be used (see below for documentation).

To install, click to download the [tar from Grafana's website][5]. Once downloaded, the contents of the file should be hosted by a webserver. After extracting the files, a simple webserver can be started by doing the following:

```bash
cd grafana-1.6.1
python -m SimpleHTTPServer $PORT
```

This creates a simple webserver. Before Grafana will display anything useful, there is some configuration that is necessary. 

In the Grafana folder, you need to rename config.sample.js to config.js:

```bash
mv config.sample.js config.js
```

Open the file:

```bash
sudo nano config.js
```

For this installation, the config.js file should look like the following:

```js
///// @scratch /configuration/config.js/1
 // == Configuration
 // config.js is where you will find the core Grafana configuration. This file contains parameter that
 // must be set before Grafana is run for the first time.
 ///
define(['settings'],
function (Settings) {
  

  return new Settings({

    // datasources, you can add multiple
    datasources: {
      graphite: {
        type: 'graphite',
        url: "http://127.0.0.1:80",
        default: true,
      },
    },

    // elasticsearch url
    // used for storing and loading dashboards, optional
    // For Basic authentication use: http://username:password@domain.com:9200
    elasticsearch: "http://127.0.0.1:9200",

    // default start dashboard
    default_route: '/dashboard/file/default.json',

    // Elasticsearch index for storing dashboards
    grafana_index: "grafana-dash",

    // timezoneOFfset:
    // If you experiance problems with zoom, it is probably caused by timezone diff between
    // your browser and the graphite-web application. timezoneOffset setting can be used to have Grafana
    // translate absolute time ranges to the graphite-web timezone.
    // Example:
    //   If TIME_ZONE in graphite-web config file local_settings.py is set to America/New_York, then set
    //   timezoneOffset to "-0500" (for UTC - 5 hours)
    // Example:
    //   If TIME_ZONE is set to UTC, set this to "0000"
    //
    timezoneOffset: "-0700",

    // set to false to disable unsaved changes warning
    unsaved_changes_warning: true,

    // set the default timespan for the playlist feature
    // Example: "1m", "1h"
    playlist_timespan: "1m",

    // Add your own custom pannels
    plugins: {
      panels: []
    }

  });
});
```

**NOTE**: Be sure to set the timezoneOffset to the correct offset. This needs to match what was set in the graphite config file `local_settings.py` otherwise the graphs will display incorrect time data. Also note that Elasticsearch is running on port 9200. The documentation for the installation of Elasticsearch will follow. 


####Configure Elasticsearch

Elasticsearch is a flexible and powerful open source, distributed, real-time search and analytics engine. Architected from the ground up for use in distributed environments where reliability and scalability are must haves, Elasticsearch gives you the ability to move easily beyond simple full-text search.

Download the [tar from Elasticsearch's website][6] and then extract. Once extracted, it can be started with the following commands:

```bash
$ bin/elasticsearch
```
Under *nix system, the command will start the process in the foreground. To run it in the background, add the -d flag to it:

```bash
$ bin/elasticsearch -d
```

In order to make this work correctly, make sure the following line is set in Grafana's `config.js` file:

```js
elasticsearch: "http://127.0.0.1:9200",
```

Replace the url with the url that you are using. Be sure that you specify it to look to port 9200. Elasticsearch is a service and can be stopped with the command:

Because of all these running services, it can become confusing on what is/isn't running. To view **all** running services, run the following command:

```bash
sudo service --status-all
```



####Configure Apache

The last thing that we have to do in order to make Grafana work with graphite is to configure Apache to work with this alternative dashboard. You need to enable the headers module as it provides directives to control and modify HTTP request and response headers. Headers can be merged, replaced or removed.

To enable the headers module, simply type:
```bash
sudo a2enmod headers
```

Because you haven’t used an alternative dashboard for graphite before you need to enable CORS (Cross Origin Resource Sharing). This is only required if Grafana is hosted on a different web domain from your graphite-web.

Open up `apache2-graphite.conf` and add the following lines right above the <location> blocks:

```
Header set Access-Control-Allow-Origin "*"
Header set Access-Control-Allow-Methods "GET, OPTIONS"
Header set Access-Control-Allow-Headers "origin, authorization, accept"
```

**Note**: using "*" leaves your graphite instance quite open so you might want to consider using "http://my.grafana.com" in place of "*". 

Save and close the file. The only thing left is to restart the apache service:

```bash
sudo service apache2 reload
```

After this, the Grafana dashboard should be configured to work with Graphite. Graphite (right now) is running on localhost:80 and this Grafana dashboard will be available on localhost:8000. For more information on Grafana and how to work with it please refer to [Grafana's Intro Guide][7].

###References

1. [wikipedia list of time zones][1]
2. [collectd's homepage][2]
3. [Etsy's StatsD][3]
4. [package statsd][4]
5. [tar from Grafana's website][5]
6. [tar from Elasticsearch's website][6]
7. [Grafana's Intro Guide][7]


[1]: http://en.wikipedia.org/wiki/List_of_tz_database_time_zones "wikipedia list of timezones"
[2]: https://collectd.org/ "collectd's homepage"
[3]: https://github.com/etsy/statsd "Etsy's StatsD"
[4]: http://godoc.org/github.com/etsy/statsd/examples/go "package statsd"
[5]: http://grafanarel.s3.amazonaws.com/grafana-1.6.1.tar.gz "tar from Grafana's website"
[6]: https://download.elasticsearch.org/elasticsearch/elasticsearch/elasticsearch-1.2.2.tar.gz "tar from Elasticsearch's website"
[7]: http://grafana.org/docs/features/intro/ "Grafana's Intro Guide"