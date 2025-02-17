## Installation
Download LogViewer from [Github releases](https://github.com/sevdokimov/log-viewer/releases) and unpack it to any folder.<br>
Make sure the machine has installed Java 8 or later.

## Usage

Run `log-viewer-1.0.5/logviewer.sh` 

Web UI will be available at http://localhost:8111. There will be a file tree where you can select a log to view. 

Also, you can open the log by the direct link _h<span>t</span>tp://localhost:8111/log?***log=$pathToLogFile***_. Several log files can be
opened in one view, pass several "log" query parameters, for example: _h<span>tt</span>p://localhost:8111/log?log=***$pathToLogFile1***&log=***$pathToLogFile2***&log=***$pathToLogFile3***_<br> 
Note: all log files must have full timestamp, otherwise LogViewer cannot merge them.

## Configuration

Configuration is located in `log-viewer-1.0.5/config.conf`, the file has [HOCON](https://github.com/lightbend/config)
format. 

#### List of available log files

A list of available log files are defined in `logs = [ ... ]` section of the configuration file. The default configuration
gives access to all files with ".log" extension:
```hocon
logs = [
  {
    path: "**/*.log"
  }
]
```

you can replace the default configuration with a more accurate configuration like

```hocon
logs = [
  {
    path: ${HOME}"/one-app/logs/*.log"
  },
  {
    path: ${HOME}"/second-app/logs/*.log"
  },
  {
    path: "/var/log/syslog"
  }
]
```

Each `{ path: "..." }` section opens access to log files by a pattern. The pattern supports wildcards "*" matches a sequence
of any characters except "/", "**" matches a sequence of any characters include "/".<br>
${HOME} will be replaced with the environment variable "HOME", it is a feature of [HOCON](https://github.com/lightbend/config#uses-of-substitutions).

#### Log format

LogViewer detects the format of log files automatically. If the format cannot be detected automatically or if you want specify
the format more detailed, you can add `format` section beside `path` definition.

In the following example all files with ".log" extension in `${HOME}"/my-app/logs` directory will be parsed as Log4J generated logs
with pattern `%date{yyyy-MM-dd_HH:mm:ss.SSS} [%thread] %-5level %logger{35} - %msg%n`  
```hocon
logs = [
  {
    path: ${HOME}"/my-app/logs/*.log"

    format = {
      type: Log4jLogFormat
      pattern: "%date{yyyy-MM-dd_HH:mm:ss.SSS} [%thread] %-5level %logger{35} - %msg%n"       
    }
  }
]
```

`format` property contains an object that defines a log format. The object must contain `type` property. Other properties
depend on format type. The configuration supports the following format types:

###### Log4J format

Log file has been generated by [Log4J](https://logging.apache.org/log4j/2.x/index.html)

```hocon
format = {
  type: Log4jLogFormat
  pattern: "%date{yyyy-MM-dd_HH:mm:ss.SSS} [%thread] %-5level %logger{35} - %msg%n"
  charset: UTF-8       
}  
```
`pattern` - Format of log lines defined the same way as in [Log4j configuration](https://logging.apache.org/log4j/2.x/manual/layouts.html#PatternLayout) <br>
`charset` - _(optional)_ file encoding name 

###### Logback format

Log file has been generated by [Logback](http://logback.qos.ch/)

```hocon    
format = {
  type: LogbackLogFormat
  pattern: "%date{yyyy-MM-dd_HH:mm:ss.SSS} [%thread] %-5level %logger{35} - %msg%n"
  charset: UTF-8       
}
```                      
`pattern` - Format of log lines defined the same way as in [Logback configuration](http://logback.qos.ch/manual/layouts.html) <br>
`charset` - _(optional)_ file encoding name 

###### Regex format

Log format can be defined with [regular expression](https://docs.oracle.com/javase/7/docs/api/java/util/regex/Pattern.html).
The log parser applies the regex to each line in the log. If a line matches regex, it is a log event,
if not, the line will be appended to a log event above the line.

```hocon    
format = {
  type: RegexLogFormat
  regex: "(?<date>\\d{4}-\\d\\d-\\d\\d_\\d\\d:\\d\\d:\\d\\d\\.\\d{3}) +(?<level>[A-Z]+) +(?<logger>[\\p{javaJavaIdentifierPart}.]+) +- (?<msg>.+)"
  charset: UTF-8
  fields: [
    { name: "date", type: "date" },
    { name: "level", type: "level/log4j" },
    { name: "logger", type: "class" },
    { name: "msg", type: "message" },
  ]
}
```                      

`regex` - a regex applying to each line

`fields` - list of log fields. Each field description has `name` and `type`. Regex must contain
[named capturing groups](https://www.logicbig.com/tutorials/core-java-tutorial/java-regular-expressions/named-captruing-groups.html)
for each field name. `type` is optional, list of available types is [here](to_be_done).
 
`charset` - _(optional)_ file encoding name