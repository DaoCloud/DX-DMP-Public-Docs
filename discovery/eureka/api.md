# Clientless SDK service discovery

For languages that are not `JVM` and have no `SDK` implementation. Eventually the simplest way to get the service table is to use the `Http API`.

## Get all registered services

```bash
curl --request GET \
  --url http://${YOUR_EUREKA_ADDRESS}/eureka/v2/apps \
```

A message in the following `XML` format will be returned：

```xml
<applications>
  <versions__delta>1</versions__delta>
  <apps__hashcode>UP_48_</apps__hashcode>
  <application>
    <name>BOOKMARK-SERVICE</name>
    <instance>
      <instanceId>192.168.1.148:bookmark-service:8762</instanceId>
      <hostName>192.168.1.148</hostName>
      <app>BOOKMARK-SERVICE</app>
      <ipAddr>192.168.1.148</ipAddr>
      <status>UP</status>
      <overriddenstatus>UNKNOWN</overriddenstatus>
      <port enabled="true">8762</port>
      <securePort enabled="false">443</securePort>
      <countryId>1</countryId>
      <dataCenterInfo class="com.netflix.appinfo.InstanceInfo$DefaultDataCenterInfo">
        <name>MyOwn</name>
      </dataCenterInfo>
      <leaseInfo>
        <renewalIntervalInSecs>30</renewalIntervalInSecs>
        <durationInSecs>90</durationInSecs>
        <registrationTimestamp>1551854657222</registrationTimestamp>
        <lastRenewalTimestamp>1551855922809</lastRenewalTimestamp>
        <evictionTimestamp>0</evictionTimestamp>
        <serviceUpTimestamp>1551854657222</serviceUpTimestamp>
      </leaseInfo>
      <metadata>
        <management.port>8762</management.port>
      </metadata>
      <homePageUrl>http://192.168.1.148:8762/</homePageUrl>
      <statusPageUrl>http://192.168.1.148:8762/info</statusPageUrl>
      <healthCheckUrl>http://192.168.1.148:8762/health</healthCheckUrl>
      <vipAddress>manage-system</vipAddress>
      <secureVipAddress>manage-system</secureVipAddress>
      <isCoordinatingDiscoveryServer>false</isCoordinatingDiscoveryServer>
      <lastUpdatedTimestamp>1551854657222</lastUpdatedTimestamp>
      <lastDirtyTimestamp>1551854657063</lastDirtyTimestamp>
      <actionType>ADDED</actionType>
    </instance>
  </application>
</applications>
```

The `<application>` tag contains the service, which consists of multiple `<instance>`s, and the `<ipAddr>` and `<port>` in the `<instance>` are the addresses of the service instances.

### Query the list of instances of a single service

```bash
curl --request GET \
  --url http://${YOUR_EUREKA_ADDRESS}/eureka/v2/apps/{YOUR_APP_ID} \
```

Results as above

### Query information about a single instance of a single service

```bash
curl --request GET \
  --url http://${YOUR_EUREKA_ADDRESS}/eureka/v2/apps/{YOUR_APP_ID}/{YOUR_INSTANCE_ID} \
```

Results as above

## Appendix

- [Eureka Restful API](https://github.com/Netflix/eureka/wiki/Eureka-REST-operations)
