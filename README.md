MySQL perfromance tuner
=======================

A plug-in based application that extracts performance data from the MySQL
system tables. The advisory plug-in modules then can make the performance
improvement suggestions based on that data.

Users can write new provider and consumer plug-ins thus extending the default
functionality.

The application was initially developed as an example for one of the
["Pro Python System Administration"](http://apress.com/book/view/9781430226055) book chapters.

You can find more information about the appliaction on [the project website](http://www.sysadminpy.com).

Application structure
---------------------

Application is split into three parts:

* The main application script (`mysql_inspector.py`), which reads in the log lines
  and calls the plug-in framework
* The plug-in manager framework (`plugin_manager.py`), which handles the module loading
  and data passing functions
* The plug in modules (`plugins/plugin_*.py`). These modules implement the data
  generation, gathering and processing functions.

Developing the plug-in modules
------------------------------

The plug-in modules can be split into two types: producers and consumers. The producer modules
obtain the data and make it available to the other plug-in modules. The consumer modules
can access the data and make advises based on it.
Below is a sample skeleton of the producer module, which only needs to implement the `generate`
method. The data is then place in the shared data structure under the plugin name space.

class ServerStatusVariables(Plugin):
    
    def __init__(self, **kwargs):
        self.keywords = ['provider']
        print self.__class__.__name__, 'initialising...'
    
    def generate(self, **kwargs):
        ...
        return result

Similarly, the consumer plug-in implements two methods: `process()` and `report()`:

    class PerformanceAdvisor(Plugin):
    
        def __init__(self, **kwargs):
            self.keywords = ['consumer']
            print self.__class__.__name__, 'initialising...'
    
        def process(self, **kwargs):
            print self.__class__.__name__, 'processing data...'
    
        def report(self, **kwargs):
            print self.__class__.__name__, 'reporting...'

Note that the type of a plug-in is determined by inspectiing the `keywords` property.
You might as well have one module implementing both types, but it's better to split
the functionality into two separate classes.

Usage
-----

When you run the main application (`mysql_inspector.py`), all plug-in modules will be 
automatically loaded in and their appropriate methods called. Make sure that you
provide the valid connection settings in the `mysql_db.cfg` configuration file.

    $ ./mysql_inspector.py
    [...]
    SlowQueriesAdvisor reporting...
    There are 0 slow requests out of total 15, which is 0.000000%
    Currently all queries taking longer than 10.000000 are considered slow
    The current slow queries ratio seems to be reasonable

