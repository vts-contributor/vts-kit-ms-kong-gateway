Logging Library for Spring Boot
-------
This library provides utilities that make it easy to add logging into spring boot project

<b>Feature List</b>
* [Application Logging](#Application-Logging)
* [Kpi Logging](#Kpi-Logging)

<b>Output type supported</b>
* Console (Default)
* File
* MariaDB (Use for KPI log)

<b>The built-in configuration</b>
* Pattern:
    * Application Log Message Pattern: `%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5p %c{1}:%X{line} - %m%n`
    * KPI Log Message Pattern: `ApplicationCode|ServiceCode|SessionId|IpPortParentNode|IpPortCurrentNode|RequestContent|ResponseContent|StartTime|EndTime|Duration|ErrorCode|ErrorDescription|TransactionStatus|ActionName|Username|Account`
    * Archived File Name Pattern: `archived-*.zip`
* File Rolling Policy:
    * Max History: `3`
    * Max File Size: `10MB`

Quick start
-------
* Just add the dependency to an existing Spring Boot project
```xml
<dependency>
    <groupId>com.atviettelsolutions</groupId>
    <artifactId>vts-kit-ms-logs-handler</artifactId>
    <version>1.0.0</version>
</dependency>
```

* Add line `<include resource="vtskit-default-logback.xml"/>` to your `logback.xml` file. Example:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="vtskit-default-logback.xml"/>
</configuration>
```

* Then, add the following properties to your `application-*.yml` file.
```yaml
vtskit:
  logs:
    level: INFO # Optional. Default is INFO
    kpi-logs:
      application-code: DEMO-APPLICATION # Default ApplicationCode value
      service-code: ${spring.application.name} # Default ServiceCode value
```

Usage
-------
### Application Logging
By default, application log will be logged to the console log.

Example code to write application log:
```java
private static final Logger LOGGER = LoggerFactory.getLogger(<Your-Class>);

// Info level
AppLogService.info(LOGGER, "message");

// Debug level
AppLogService.debug(LOGGER, "message");

// Warning level
AppLogService.warn(LOGGER, "message");

// Error level
AppLogService.error(LOGGER, "error");
```

To configure logging to the file, add below configuration to `application-*.yml` file:
```yaml
vtskit:
  logs:
    log-folder: logs # Folder to save log file
    app-logs:
      error-log-file-name: ${spring.application.name}.error.log # File to save error log
      console-log-file-name: ${spring.application.name}.log # File to save all log
```

### Kpi Logging
#### Automation
By default, All requests will be automatically saved kpi log.

To limit by url patterns, add below configuration to `application-*.yml` file:
```yaml
vtskit:
  logs:
    kpi-logs:
      allow-url-patterns: /**/getList,/**/update # Default is '/**' allow all requests
```

To disable automation mode, add below configuration:

```yaml
vtskit:
  logs:
    kpi-logs:
      allow-url-patterns: ignore
```

#### Manual
If you want to handle saving kpi log manually, follow the steps below

<b>Step 1</b>: Define `KpiLogService` instance:
```java
private KpiLogService kpiLogService;

@Autowired
public void setKpiLogService(KpiLogService kpiLogService) {
    this.kpiLogService = kpiLogService;
}
```

<b>Step 2</b>: Set KPI Log information using `KpiLog.Builder`:
```java
KpiLog.Builder builder = new KpiLog.Builder();
builder.sessionId("SessionId");
builder.ipPortParentNode("127.0.0.1");
builder.ipPortCurrentNode("127.0.0.1");
builder.requestContent("requestContent");
builder.responseContent("responseContent");
builder.startTime(new Date());
builder.errorCode("00");
builder.errorDescription("");
builder.transactionStatus(TransactionStatus.SUCCESS);
builder.actionName("actionName");
builder.username("username");
builder.account("account");
builder.endTime(new Date());
```

<b>Step 3</b>: Execute save KPI Log
```java
kpiLogService.writeLog(LOGGER, builder.build());
```

#### Output configuration

By default, KPI log will be logged to the console log. In addition, it can be configured to write to file or database.

To configure logging to the file, add below configuration to `application-*.yml` file:
```yaml
vtskit:
  logs:
    log-folder: logs # Folder to save log file
    kpi-logs:
      kpi-log-file-name: ${spring.application.name}.kpi.log # File to save kpi log
```

Similarly, to save the kpi log to the database, add below configuration to `application-*.yml` file:
```yaml
vtskit:
  logs:
    kpi-logs:
      datasource: # Configuration Maria DB for store kpi log
        url: jdbc:mariadb://localhost:3307/test-database
        username: root
        password: root
```
The system will automatically create a table named `KPI_LOG` and save the kpi log data there.

Contribute
-------
#### Setting up the development environment
* <b>IDE:</b> Eclipse, Intellij IDEA
* <b>JDK:</b> >= JDK 8
* <b>Maven:</b> >= 3.6.0
* <b>Build:</b>
```shell script
mvn clean install
# Skip Unittest
mvn clean install -DskipTests
```
#### Contribute Guidelines
If you have any ideas, just open an issue and tell us what you think.

If you'd like to contribute, please refer [Contributors Guide](CONTRIBUTING.md)

License
-------
This code is under the [MIT License](https://opensource.org/licenses/MIT).

See the [LICENSE](LICENSE) file for required notices and attributions.
