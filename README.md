# CA APM Monitor 4.0 for IBM WebSphere DataPower

# Description

The CA APM Monitor 4.0 for IBM WebSphere DataPower (DataPower Monitor) agent provides essential IBM WebSphere DataPower appliance performance data. Using Investigator and WebView, you view the data about the DataPower appliance and services deployed on the appliance. The Team Center map displays the DataPower appliance as a component correlated to application components calling into and from the appliance.

When a web service transaction has a performance issue, DataPower Monitor quickly correlates the performance data from a variety of sources to locate the problem.

Correlation occurs for the DataPower appliances, backend applications, and any of the components participating in your Enterprise Service Bus architecture.

# Short Description
The CA APM Monitor 4.0 for IBM WebSphere DataPower agent provides essential IBM WebSphere DataPower appliance performance data.

# APM version
CA APM 10.5.1 and later

# Dependencies
Enterprise Manager 10.5.1 and later

Tested with IBM DataPower Gateway Firmware:IDG.7.5.0.0 and Build:274960.

Other DataPower versions should also work. If you have problems, contact [CA Support](https://support.ca.com).

# Prerequisites
Turn on the DataPower appliance XML Management interface.

1. In the DataPower Gateway administration left navigation pane, click **Network**, then **XML Management Interface**.
2. To set the Administrative state, choose the **enabled** button.
3. To set Enabled services, select these boxes: **SOAP management URI**, **SOAP configuration management**, and **SOAP configuration management (v2004)**.

# Install DataPower Monitor

1.  Download DataPower Monitor from the [CA APM Github](https://github.com/CA-APM/Datapower/releases).
2.  Navigate to the downloaded DataPower Monitor.
3.	Unzip `DatapowerMonitor-4.0.0.zip` into the target installation directory, which we will reference as &lt;*DataPower_Home*&gt;.
4.	Copy `Datapower_MM.jar` from &lt;*DataPower_Home*&gt;/modules to the &lt;*MOM_Home*&gt;/deploy directory.

# Install the Server SSL Certificate
The IBM WebSphere SOA Appliance DataPower XML Management Interface requires basic credentials over SSL authentication.

To provide the credentials, install the DataPower SSL server certificate into the JRE trusted certificate store.

1. From a command prompt in the &lt;*DataPower_Home*&gt;/bin directory, run the apppropriate certificate installation script:
  * Windows: `InstallCert.cmd <host>[:port] [keystore] [passphrase]`
  * Linux `InstallCert.sh <host>[:port] [keystore] [passphrase]`

  Where:
   - Host: Hostname or IP address of the DataPower appliance
   - Port: Port on which the DataPower appliance is listening
   - Keystore: Trusted keystore of the JRE that runs the DataPower monitor  
     The default value is `<JRE_Home>/lib/security/cacerts`.
   - Passphrase: Passphrase protecting the keystore  
     The default value is `changeit`.
2. Press **1** to install the first certificate in the certificate list.

**Example**

1. The DataPower appliance IP address is 10.160.160.33 and the port is 9090.
2. You run the command: `InstallCert 10.160.160.33:9090`.  
   A list of three certificates appears.
3. You press **1** to install the certificate.

**More information:** See the DataPower monitor user guidev4.0.pdf in the &lt;*DataPower_Home*&gt;/doc directory.

# Configure DataPower Monitor

1. Configure the datapower.properties file located in the &lt;*DataPower_Home*&gt;/config directory.

  a. (Optional) Configure the log file location.  
	 The default is DatapowerMonitor/logs/DataPowerMonitor.log.

  b. (Optional) Configure the logging level.  
	 The default is `INFO`.  
	 Here is an example of setting `debug` level:

		log4j.logger.com.wily.field.dpmon=DEBUG,console,logfile

  c. Specify the Enterprise Manager to which the agent sends the performance data. For example,

		agentManager.url.1=<EM_Host>:5001

  d. Specify the SSL Context supported by your DataPower installation. For example,

		sslContext=TLSv1.2

2. Configure properties in the DatapowerMonitor-config.xml file, located in the &lt;*DataPower_Home*&gt;/config directory.

  a. Specify the DataPower connection and authentication properties. For example,

		<properties>
			<property name="host">IPAddress</property>
			<property name="port">5550</property>
			<property name="path">service/mgmt/2004</property>
			<property name="usr">admin</property>
			<property name="pwd">password</property>
		</properties>

  b. To monitor more than one DataPower appliance, add extra entries to the &lt;devices&gt; element. Be sure that you use the same values that are already defined in the &lt;properties&gt; element.

  Here is an example where the `usr` (user) and `pwd` (password) properties shown in the Step 2.a. example code block &lt;properties&gt; element are used again in the &lt;devices&gt; element.

		<devices>
				<device name="DC-East">
					<url>https://${host_east}:${port}/${path}</url>
					<user>${usr}</user>
					<password>${pwd}</password>
				</device>
				<device name="DC-West">
					<url>https://${host_west}:${port}/${path}</url>
					<user>${usr}</user>
					<password>${pwd}</password>
				</device>
		</devices>

  c. Configure the &lt;domains&gt; element values.

  Add the DataPower appliance domains that you want to monitor to the &lt;domains&gt; element. The &lt;domains&gt; element is located within the &lt;engine&gt; element. Be sure that the `domain name` value matches a domain within DataPower.

  DatapowerMonitor-config.xml includes three pre-defined domains: `default`, `HelloDP`, and `prod`. DataPower Monitor ignores domains that are not used.

  Here is an example &lt;domains&gt; element:

		<domains>
		 <domain name="default">
		  <val>default</val>
		 </domain>
		 <domain name="HelloDP">
		  <val>HelloDP</val>
		 </domain>
		 <domain name="prod">
		   <val>prod</val>
		 </domain>
		</domains> ..

	d. Configure properties to change, add, or delete the metrics that DataPower Monitor provides.

	You can define extra metrics by adding &lt;status&gt; elements to the &lt;metric-set&gt; element. The `status name` and `metric statusProperty` values for a new &lt;metric-set&gt; element must match the DataPower XML Management interface `class` value.

	In this example, the status name `MemoryStatus` must exist in the DataPower XML Management interface with at least the `TotalMemory`, `UsedMemory`, and `Usage` values for metric statusProperty property instances.

		<status name="MemoryStatus">
            <prefix>Memory:</prefix>
            <metric statusProperty="TotalMemory" type="LongAverage">Total [bytes]</metric>
            <metric statusProperty="UsedMemory" type="LongAverage">Used [bytes]</metric>
            <metric statusProperty="Usage" type="IntAverage">Usage [%]</metric>
        </status>

	e. Configure engines.

	An engine corresponds to a thread that polls for DataPower appliance metrics. In DatapowerMonitor-config.xml, each &lt;engine&gt; element includes references to a &lt;domains&gt;, a &lt;device&gt; and several &lt;metric-set&gt; elements. Each of these elements provide a `ref` value for the reference. Make sure that the `ref` values are valid DataPower appliance domain, device, and metric-set elements.

	We recommend defining one engine for each device and domain. For example, if you monitor two DataPower appliances with three domains each, you define six engines.

	Here is an example definition for the first engine:

		<engine name="E1" enabled="true" retainValues="false">
		    <device ref="HelloDP"/>  
		    <domain ref="default"/>
		    <metric-set ref="Basic">DeviceMetrics|</metric-set>
		    <metric-set ref="Static">DeviceMetrics|</metric-set>
		    <interval unit="second">15</interval>
		</engine>

# Upgrade DataPower Monitor
You can upgrade CA APM Monitor 3.2 for IBM WebSphere DataPower to CA APM Monitor 4.0 for IBM WebSphere DataPower.

**Follow these steps:**

1. Copy the CA APM Monitor 3.2 for IBM WebSphere DataPower properties files.  
 a. Navigate to the &lt;*DataPower_Home*&gt;/config directory.  
 b. Copy these files and save them in a non-IBM WebSphere DataPower 3.2 directory:   
 *  datapower.properties  
 * DatapowerMonitor-config.xml
2. Download CA APM Monitor 4.0 for IBM WebSphere DataPower from the [CA APM Github](https://github.com/CA-APM/Datapower/releases)..
3. Navigate to the downloaded DataPower Monitor.
4. Unzip DapowerMonitor.zip into a target installation directory that is different from the CA APM Monitor 3.2 for IBM WebSphere DataPower &lt;*DataPower_Home*&gt; directory.  
   The name of the CA APM Monitor 4.0 for IBM WebSphere DataPower is now the &lt;*DataPower_Home*&gt; directory.
5. Copy Datapower_MM.jar from &lt;*DataPower_Home*&gt;/modules to the &lt;*MOM_Home*&gt;/deploy directory.
6. Configure properties in the datapower.properties file.  
 a. Find the copy of the CA APM Monitor 3.2 for IBM WebSphere DataPower datapower.properties file.  
 b. Navigate to the CA APM Monitor 4.0 for IBM WebSphere DataPower datapower.properties file located in the &lt;*DataPower_Home*&gt;/config directory.  
 c. Manually copy configured properties in the CA APM Monitor 3.2 datapower.properties file to the CA APM Monitor 4.0 datapower.properties file.  
 d. Provide the Enterprise Manager host and port values in the CA APM Monitor 4.0 datapower.properties file.  
7. Configure properties in the DatapowerMonitor-config.xml file.  
 a. Find the copy of the CA APM Monitor 3.2 for IBM WebSphere DataPower DatapowerMonitor-config.xml file.  
 b. Navigate to the CA APM Monitor 4.0 for IBM WebSphere DataPower DatapowerMonitor-config.xml file located in the &lt;*DataPower_Home*&gt;/config directory.  
 c. Manually copy the property settings from the CA APM Monitor 3.2 to CA APM Monitor 4.0 DatapowerMonitor-config.xml file.  
   **Important!** Do not continue to use the CA APM Monitor 3.2 for IBM WebSphere DataPower DatapowerMonitor-config.xml file, because the CA APM Monitor 4.0 Team Center properties and metrics are not included.  
   There are new properties in the CA APM Monitor 4.0 DatapowerMonitor-config.xml that allow Team Center to display specific metrics.  
 d. Copy these elements from CA APM Monitor 3.2 to CA APM Monitor 4.0 DatapowerMonitor-config.xml:  
    * Properties  
    * Devices  
    * Domains  
    * Engines  
    * Additional metric-sets

# Use DataPower Monitor

## Start the DataPower Monitor
At a command prompt, run the appropriate startup script from the &lt;*DataPower_Home*&gt;/bin directory:

* Windows: `DatapowerMonitor.cmd`

* Linux: `./DatapowerMonitor.sh`

## Metrics
DataPower documentation includes information about the [IBM WebSphere DataPower SOA
Appliance XML Management Interface](http://www.redbooks.ibm.com/redpapers/pdfs/redp4446.pdf) that provides metric data.

## Custom Management Module
DataPower_MM.jar contains a metric grouping, dashboards, metrics, and alerts.

## Team Center Map
The DataPower appliance appears as a component on the Team Center Map. Web service calls to and from the DataPower appliance that originate from or are going to other monitored applications are correlated with the DataPower appliance. The other applications can be monitored by for example, the CA APM Java, .NET, or Node.js agents.

# Debug and Troubleshoot the DataPower Monitor

1. Set the `DEBUG` logging level.

  a. Navigate to the datapower.properties file, located in the &lt;*DataPower_Home*&gt;/config directory.

  b. Set the the DataPower Monitoring log level to `DEBUG` in the `log4j.logger.com.wily.field.dpmon` property.

      log4j.logger.com.wily.field.dpmon=DEBUG, console, logfile

2. Make sure these properties values are valid in the DatapowerMonitor-config.xml properties file:

		<properties>
			<property name="host">IPAddress</property>
			<property name="port">5550</property>
			<property name="path">service/mgmt/2004</property>
			<property name="usr">admin</property>
			<property name="pwd">password</property>
		</properties>

3. If a metric is missing in CA APM, make sure that the corresponding metric set is referenced within a  DatapowerMonitor-config.xml engine element.

4. If CA APM metric values seem wrong, compare them with the values in DataPower user interface at http://IPAddress:port/status/{classname}.

# Limitations

* A problem occurs when the DataPower Monitor agent identifies a call to a DataPower appliance as a socket call (`System hostname on port nnnn`), not a web service call. This situation causes the Team Center map to display the DataPower appliance as not connected to the calling application.
* The Team Center map does not display a DataPower appliance in these situations:  
  * When the DataPower appliance is not called directly from a Java or a Node.js application that is monitored by an APM agent. This situation includes when the DataPower appliance is the left-most component other than Business Transactions on the Team Center map.  
  * When there are multiple hops between the DataPower appliance and the called monitored application. For example, when one Datapower appliance invokes another DataPower appliance, and then invokes a monitored application.

# Known Issue

**Symptom:**

The DataPower Monitor HTTPS connections to the Datapower XML Management Interface fail with a runtime exception. A defect in the IBM JDK JSSE implementation causes the failure.

Here are the first lines of the error message:

	java.lang.RuntimeException: I/O error during HTTP(S) invocation
			at com.wily.ps.dpmon.rt.TransporterImpl.execute(TransporterImpl.java:67)
			at com.wily.ps.dpmon.rt.StandardInvoker.execute(StandardInvoker.java:33)
			at com.wily.ps.dpmon.rt.CachingInvoker.execute(CachingInvoker.java:33)
			at
	com.wily.ps.dpmon.DataPowerMonitor$InvokerWrapper.run(DataPowerMonitor.java:1
	03)

**Solution:**

The workaround is to use the SUN JDK or to upgrade the JSSE version to 1.0.3 or later.

## License
This field pack is provided under the [Eclipse Public License, Version 2.0](https://github.com/CA-APM/Datapower/blob/master/LICENSE).

## Support
This document and associated tools are made available from CA Technologies as examples and provided at no charge as a courtesy to the CA APM Community
at large. This resource may require modification for use in your environment. However, please note that this resource is not supported by CA Technologies,
and inclusion in this site should not be construed to be an endorsement or recommendation by CA Technologies. These utilities are not covered by the
CA Technologies software license agreement and there is no explicit or implied warranty from CA Technologies. They can be used and distributed freely
amongst the CA APM Community, but not sold. As such, they are unsupported software, provided as is without warranty of any kind, express or implied,
including but not limited to warranties of merchantability and fitness for a particular purpose. CA Technologies does not warrant that this resource
will meet your requirements or that the operation of the resource will be uninterrupted or error free or that any defects will be corrected. The use
of this resource implies that you understand and agree to the terms listed herein.

Although these utilities are unsupported, please let us know if you have any problems or questions by adding a comment to the CA APM Community Site area where the resource is located, so that the Author(s) may attempt to address the issue or question.

Unless explicitly stated otherwise this field pack is only supported on the same platforms as the APM core agent. See [APM Compatibility Guide](https://techdocs.broadcom.com/us/product-content/status/compatibility-matrix/application-performance-management-compatibility-guide.html).


### Support URL
https://github.com/CA-APM/Datapower/issues

# Categories
Middleware/ESB Integration
