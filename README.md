## Fuse Online Sample Extension

The repository contains extension that can be used in a integration to demonstrate the following flow:
```
 JMS -> Extension -> Datamapper -> Todo API
```


This sample integration connects to an AMQ broker to obtain item delivery records for a hypothetical enterprise.
The integration then executes a custom step that operates on the records to identify any items that were damaged when they were received.
After a simple data mapping, the integration connects to a REST API to obtain and provide contact information for vendors of damaged items.

#### Build from master

Prepared jar can be consumed from [releases](https://github.com/syndesisio/fuse-online-sample-extension/releases) page.

Alternatively it can be also build from master with corresponding **Fuse Online GA** Maven version.

Version can be obtained from Support page `https://<Fuse Online URL>/support`.

```
./mvnw -s configuration/settings.xml clean package -Dsyndesis.version=<Fuse Online version>
```


#### Camel route
```xml
<routes xmlns="http://camel.apache.org/schema/spring" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://camel.apache.org/schema/spring http://camel.apache.org/schema/spring/camel-spring.xsd">
  <route id="damage-report">
	<from uri="direct:damage-report"/>
	<log message="Received item list:\n${in.body}" />
	<unmarshal>
	  <jacksonxml unmarshalTypeName="io.syndesis.extension.InventoryList" />
	</unmarshal>
	<bean beanType="io.syndesis.extension.ReportService" method="generateReport" />
	<when>
	  <simple>${in.body} == null</simple>
	  <log message="No damaged item found."/>
	  <stop/>
	</when>
	<!-- Continue otherwise -->
    <log message="Generated report: ${in.body}"/>
  </route>
</routes>
```

#### XML format
```xml
<?xml version="1.0" encoding="UTF-8"?>
<inventoryReceived>
  <item id="XYZ123" damaged="false" vendor="Good Inc."/>
  <item id="ABC789" damaged="true" vendor="Bad Inc."/>
</inventoryReceived>
```

#### Todo Report
```
Task: Contact Joe Doe, 987 654 321. Damaged items: ABC789.
```

