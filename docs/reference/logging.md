# Logging

Kache supports efficient structured logging with different output locations and formats:

* [Console](#console)
* [Log file](#file)
* [JSON](#json)

### Console

Pretty logging to the console is supported and activated by default.

=== "YAML"
  ``` yaml
  logging:
    # Set level to debug.
    level: debug
    # Activate color log output.
    color: true # disabled by default
  ```

### File

To use structured file-based logging, just configure a log location.

=== "YAML"
  ``` yaml

  logging:
    level: info
    
    # Activate file-based logging.
    file: /var/log/kache/kache.log

    # Configure file-based logging.
    max_size:    500 # 500 megabytes
    max_backups: 3 
    max_age:     28 # days

  ```

??? info
    Kache opens or creates the logfile on first write. If the file exists and is less than MaxSize megabytes, it will open and append to that file. If the file exists and its size is >= MaxSize megabytes, the file is renamed by putting the current time in a timestamp in the name immediately before the file's extension (or the end of the filename if there's no extension). A new log file is then created using original filename.
    
    Whenever a write would cause the current log file exceed MaxSize megabytes, the current file is closed, renamed, and a new log file created with the original name. Thus, the filename you give Logger is always the "current" log file.

    Backups use the log file name given to Logger, in the form name-timestamp.ext where name is the filename without the extension, timestamp is the time at which the log was rotated formatted with the time.Time format of 2006-01-02T15-04-05.000 and the extension is the original extension. For example, if your Logger.Filename is /var/log/foo/server.log, a backup created at 6:30pm on Nov 11 2016 would use the filename /var/log/foo/server-2016-11-04T18-30-00.000.log

    [More..](https://pkg.go.dev/gopkg.in/natefinch/lumberjack.v1#Logger)


### JSON 

To log in JSON format, just change the format of the log output to `json`. This works with both, console and file logging.

=== "YAML"
  ``` yaml
  logging:
    # Set level to debug.
    level: debug
    # Set format to json.
    format: json
  ```

### Reference

| Directive     | Type        | Description                          |
| -----------   | ----------- | ------------------------------------ |
| `level`       | `string`    | Specifies the log level. Supported log levels are: `trace`, `debug`, `info`, `warn`, `error`, `fatal`, `panic`. |
| `format`      | `string`    | Specifies the log format. Default is console, `json` changes log output to JSON format. |
| `color`       | `bool`      | Enables colored log output. Disabled by default. Logging to files never colors the output. |
| `file`        | `string`    | Sets the log file path. |
| `max_size`    | `int`       | Sets the maximum size in megabytes of the log file before it gets rotated. It defaults to 100 megabytes. |
| `max_age`     | `int`       | Sets the maximum number of days to retain old log files based on the timestamp encoded in their filename.  Note that a day is defined as 24 hours and may not exactly correspond to calendar days due to daylight savings, leap seconds, etc. The default is not to remove old log files based on age. |
| `max_backups` | `int`       | Sets the maximum number of old log files to retain.  The default `s to retain all old log files (though MaxAge may still cause them to get deleted.) |
| `compress`    | `bool`      | Specifies if the rotated log files should be compressed using gzip. The default is not to perform compression. |
