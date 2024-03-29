# Welcome to fae-util 

## Overview 

* The utility is designed to support spidering a website or traversing a list of URLs and analyzing the content of each URL using !JavaScript. 
* Utility accepts a set of URLs and a text file containing a list of JavaScript files to be used for analysis. 
* Using these inputs, it retrieves the DOM specified by the URL information, runs the code in the JavaScript files using the DOM as an input, and returns the output of the !JavaScript. 
* The !JavaScript files used for analysis must return text content (e.g. in some format like JSON or XML) for each URL and this content is saved to disk.
* The utility also provides additional information URL, such as the request and returned URL, and URLS that were filtered or could not be processed for some reason (i.e. broken link, invalid script)

## fae-util Command Line Options 

### Specifying URLs to Analyze 

* If -u or -- url is specified, its value is considered to be the starting URL of the analysis.
* If -m or --multipleUrls is specified, its value points to a file containing a list of URLs to traverse.
* If -m (or --multipleUrls) is specified along with -d, --depth, -s, --spanDomains, -i, --includeDomains or -e, --excludeDomains, exit with an error message: "Cannot specify <options> with --multipleUrls."
* If both -u (or --url) and -m (or --multipleUrls are specified, exit with an error message: "Cannot specify both starting URL and multiple URLs file."

```
-u, --url <arg>              Required (unless -m, --multipleUrls is specified): starting URL
# or
-m, --multipleUrls <arg>     Required (unless -u, --url is specified): filename containing URLs to evaluate
```

### Other Command Line Options 

```
-o, --outputDirectory <arg>  Required: directory for results files

-p, --path                 Optional: path the URL must include to be included in the evaluation


-c, --config <arg>           Optional: filename of configuration parameters
-a, --authorization <arg>    Optional: filename of authorization information

-s, --spanDomains <arg>      Optional: traverse the subdomains of these domains (comma-separated list), in addition to the domain specified by the URL
-i, --includeDomains <arg>   Optional: traverse these domains (comma-separated list) in addition to the domain specified by the URL
-e, --excludeDomains <arg>   Optional: do not traverse these domains (comma-separated list; valid only if -s is specified; each domain must be a subdomain of an entry in spanDomains)

-d, --depth <arg>            Optional: maximium depth to traverse (number: 1 | 2 | 3, default = 1, which means no traversing)
-w, --wait <arg>             Optional: maximium time in milliseconds to wait when processing a page, default = 30000 msec. (30 seconds)

-r, --ruleset <arg>          Optional: OAA ruleset ID ('ARIA_TRANS' | 'ARIA_STRICT', default = 'ARIA_TRANS')
-xo, --exportOption <arg>    Optional: True | False, default = False 

-v, --version                Optional: output the fae-util version number
-h, --help                   Optional: output help for command-line syntax and options

-m, -maxPages <arg>          Optional: maximum number of pages to process, default is no limit

-j, --javaScript             Optional: HtmlUnit javascript option (true | false, default = true, which enables javascript)  
```

### Command Line Options with NO Changes

```
-D, --debug <arg>            Optional: turn on debugging output
-V, --verbose <arg>          Optional: turn on HtmlUnit logging output
```

### Properties File Format 

The properties file defines default values for some user-accessible options, but also includes properties that are not user-editable.

```
browserVersion=<string: 'Firefox' | 'Chrome' | 'IE' | 'Edge' | 'Best', default='Firefox'> (the latest version of each browser type supported in HTMLUnit is assumed, see Reference 1)
scripts=<filename, default='openajax_a11y/oaa_a11y_scripts.txt'>
excludeExtensions=<filename>
wait=<number in milliseconds, default=30000>

ruleset=<ruleset id, default='ARIA_TRANS'>

exportFunction=<string, potential options "toJSON", "toXML", "toHTML", "toDjango", default="toJSON">
exportExtension=<string, potential options "json", "xml", "html", "py", default='json'>
exportOption=<True | False, default=False>
```

Reference 1: [http://htmlunit.sourceforge.net/apidocs/com/gargoylesoftware/htmlunit/BrowserVersion.html * HTMLUnit Browser Constants]

### Configuration File Format 

The configuration file allows users to override some values in the properties file (e.g., wait time, ruleset, etc.), as well as to specify options such as spanDomains which may be relatively permanent (i.e. used across multiple evaluations).

NOTE: Options specified on the command-line override settings in the configuration file. fae-util should output messages stating which command-line options overrode which config file settings just to make this explicit.

```
url=<startingURL>
# or (may only specify one of these)
multipleUrls=<filename>

path=<string>
javaScript=<true | false>

spanDomains=<domain1>,<domain2>,...
includeDomains=<domain1>,<domain2>,...
excludeDomains=<domain1>,<domain2>,...

depth=<number 1 | 2 | 3, default=1>
wait=<number in milliseconds, default=30000>

ruleset=<ruleset id, default='ARIA_TRANS'>

exportFunction=<string, default="toJSON">
exportExtension=<string, default='json'>
exportOption=<boolean, default='False'>

excludeScripts=<script1>,<script2>,...
``` 

### OAA Configuration Script File

* This file is generated by fae-util for each request for analysis
* The configuration information is from 6 items in the above configuration file 
* File can be placed in the output directory and named "oaa_a11y_config.js"
* This file is executed after all other !OpenAjax scripts have been loaded

```
var doc = window.document;

var ruleset = OpenAjax.a11y.all_rulesets.getRuleset("<ruleset>"); 
  
var evaluation = ruleset.evaluate(doc.location.href, doc.title, doc, null, true);

var out = evaluation.<oaaExportFunction>(<exportOption>);

out;
```

### OAA Authorization File Format 

* This file provides authetication information for set of websites 
* Support for:
  * Login anchors
  * Login forms
  * Login fields
  * Partial urls
  

```
<?xml version="1.0" encoding="UTF-8"?>
<authorizations> 

  <!-- url with login anchor -->
  <authorization>
    <url>https://some-university.edu/cf/index.cfm</url> 
    <!-- need to indicate anchor to look for to go to login page, optional-->
    <anchor text="Login"/> 
    <form name="Login"/> 
    <submit name="Login"/>
    <control name="USER" value=""/> 
    <control name="PASSWORD" value=""/> 
  </authorization> 

  <!-- full login url -->
  <authorization> 
    <url>https://some-university.edu/frontEnd/index.jsp</url>
    <form id="easForm"/> 
    <submit value="Login"/> 
    <control name="inputEnterpriseId" value=""/> 
    <control name="password" value=""/>
    <!-- optional -->
    <verification enabled="true" text="Sign Out"/>
  </authorization> 

  <!-- login url starting with -->
  <authorization> 
    <url>https://some-university.edu/ParamFrontE</url> 
    <url-match type="startsWith"/> 
    <form name="Login"/>
    <!-- submit without a name or value --> 
    <submit tag="button" type="submit"/> 
    <control name="USER" value=""/> 
    <control name="PASSWORD" value=""/> 
    <verification enabled="false" text="Log Out"/>
  </authorization>

  <!-- login url with more than two login fields -->
  <authorization> 
    <url>https://some-university.edu/StartFrontEn</url> 
    <!-- form without a name or id -->
    <form index="0"/> 
    <submit tag="input" type="image"/> 
    <control count="3" />
    <control id="1" name="org" value="openaccessibilityalliance" />
    <control id="2" name="login" value="" />
    <control id="3" name="password" value="" />
    <verification enabled="true" text="Log Me Out" /> 
  </authorization> 

</authorizations>

```
## fae-util Examples ##

### Example 1: Span domains that include all first-level subdomains

* Follow all links in the domain specified by the URL (illinois.edu) and all of its first-level subdomains (*.illinois.edu)
* This example will traverse illinois.edu, calendar.illinois.edu, cita.illinois.edu, www.illinois.edu, etc.

```
fae-util -u http://illinois.edu -d 2 -s illinois.edu -o temp1
```

### Example 2: Span domains that include sibling subdomains 

* Follow all links in the domain specified by the URL (cita.illinois.edu) and all of its sibling domains (including illinois.edu).
* This example will traverse illinois.edu, calendar.illinois.edu and disability.illinois.edu, but will not traverse second-level subdomains such as fae.cita.illinois.edu or presentations.cita.illinois.edu.

```
fae-util -u http://cita.illinois.edu -d 2 -s illinois.edu -o temp1
```

### Example 3: Include another domain 

* Follow all links in the domain specified by the URL (illinois.edu) along with an additional domain (www.illinois.edu).
* This example will traverse illinois.edu and www.illinois.edu only.

```
fae-util -u http://illinois.edu -d 2 -i www.illinois.edu -o temp1
```

### Example 4: Span domains and exclude a subdomain ===

* Follow all links in the domain specified by the URL and all of its first-level subdomains, with one exception.
* This example will traverse cita.illinois.edu, courses.cita.illinois.edu, fae.cita.illinois.edu, but will not traverse presentations.cita.illinois.edu.

```
fae-util -u http://cita.illinois.edu -d 2 -s cita.illinois.edu -e presentations.cita.illinois.edu -o temp1
```

### Example 5: Span multiple domains and their subdomains 

* Follow all links in the domain specified by the URL (illinois.edu) as well as the domains listed after "-s" and all of their first-level subdomains.
* This example will traverse illinois.edu and news.illinois.edu, www.news.illinois.edu, aces.calendar.illinois.edu, www.inclusiveillinois.illinois.edu.

```
fae-util -u http://illinois.edu -d 2 -s news.illinois.edu,calendar.illinois.edu,inclusiveillinois.illinois.edu -o temp1
```

### Example 6: Include multiple domains (but not their subdomains) 

* Follow all links in the domain specified by the URL (illinois.edu) as well as the domains listed after "-i".
* This example will traverse illinois.edu and news.illinois.edu, calendar.illinois.edu and inclusiveillinois.illinois.edu.

```
fae-util -u http://illinois.edu -d 2 -i news.illinois.edu,calendar.illinois.edu,inclusiveillinois.illinois.edu -o temp1
```

### Example 7: Span domains and exclude multiple subdomains

* Follow all links in the domain specified by the URL and all of its first-level subdomains (as specified with "-s"), except for the domains listed after "-e".
* This example will traverse illinois.edu, news.illinois.edu, cita.illinois.edu (but not presentations.cita.illinois.edu) and all other subdomains of illinois.edu with the exception of library.illinois.edu, aces.illinois.edu, las.illinois.edu and engineering.illinois.edu.

```
fae-util -u http://illinois.edu -d 2 -s illinois.edu -e library.illinois.edu,aces.illinois.edu,las.illinois.edu,engineering.illinois.edu -o temp1
```

### Example 8: Error condition 

* If the exclude option is not a subdomain of one of the span domain options, then the exclude domain name is ignored and a message should be sent to the console indicating the error.
* In this example the -e option would be ignored.

```
fae-util -u http://illinois.edu -d 2 -s illinois.edu -e illinois.edu -o temp1
```

### Example 9: Error condition 

* If any of the -s, -i or -e options are used and -d is not specified with a value of either 2 or 3, a message should be displayed indicating the error.
* In this example, the -s and -e options would be ignored.

```
fae-util -u http://cita.illinois.edu -s cita.illinois.edu -e presentations.cita.illinois.edu -o temp1
```
