---

title: "Update from 7.7 to 7.8"
weight: 55

menu:
  main:
    name: "7.7 to 7.8"
    identifier: "migration-guide-77"
    parent: "migration-guide-minor"
    pre: "Update from `7.7.x` to `7.8.0`."

---

This document guides you through the update from Camunda BPM `7.7.x` to `7.8.0`. It covers these use cases:

# REST API Date Format

This section is applicable if you use the Camunda engine REST API.

The default date format used in the REST API requests and responses has changed from `yyyy-MM-dd'T'HH:mm:ss` to `yyyy-MM-dd'T'HH:mm:ss.SSSZ` (now includes second fractions and timezone). 
The Camunda webapps support the new format by default.

In case some custom REST clients rely on the old date format, choose one of the two following options:

1. Update REST clients to use the new format.
2. Configure custom date format for Camunda REST API (explained in detail in the [Custom Date Format]({{< relref "reference/rest/overview/date-format.md" >}})) section.

# Failed Jobs Retry Configuration

There is no need any more of enabling the retry time cycle configuration for failed jobs. This is the default behaviour from now on.

You should clean up the following lines from the process engine configuration file in order to be consistent with this behaviour.

```xml
<bean id="processEngineConfiguration" class="org.camunda.bpm.engine.impl.cfg.StandaloneInMemProcessEngineConfiguration">
  ...
  <!-- remove these properties -->
  <property name="customPostBPMNParseListeners">
    <list>
      <bean class="org.camunda.bpm.engine.impl.bpmn.parser.FoxFailedJobParseListener" />
    </list>
  </property>
  
  <property name="failedJobCommandFactory" ref="foxFailedJobCommandFactory" />
  ...
</bean>
<!-- remove this bean -->
<bean id="foxFailedJobCommandFactory" class="org.camunda.bpm.engine.impl.jobexecutor.FoxFailedJobCommandFactory" />
```

# Incident Handler

This section concerns Java API and the interface `org.camunda.bpm.engine.impl.incident.IncidentHandler` that is part of internal API.

The return type of `IncidentHandler#handleIncident` has changed from `void` to `Incident`. The API expects that an incident is created and returned.

In case there are custom incident handlers implementing that interface, the method `handleIncident(...)` should be adjusted. The method can return `null` value.

# Batch processing for database operations

Starting from version 7.8 Camunda uses batch processing to execute SQL statements over the database. Old (not batch) processing mode can be configured like this:
```xml
 <property name="jdbcBatchProcessing" value="false"/>
```

You might consider disabling the batch mode in following cases:

1. Batch processing is not working for Oracle versions earlier than 12. If you're using one of these versions you would need to disable batch processing
 in Camunda configuration to switch back to the old simple mode.

2. Statement timeout (configured by `jdbcStatementTimeout` parameter) is not working in combination with batch mode on MariaDB and DB2 databases.
So if you're using `jdbcStatementTimeout` configuration on the listed databases, consider disabling the batch mode.