# iAppLX MSDA for nacos

This iApp is an example of MSDA for nacos, including an audit processor.  

## Build (requires rpmbuild)

  Check the modules dependency:
```bash
    $ cd src/nodejs
    $ npm list
```
  Fix all dependent modules with `npm install` command.
  Go back to the root direct of the project, build an rpm package:
```bash
    $ cd ../..
    $ npm run build
```

Build output is an RPM package
## Using IAppLX from BIG-IP UI
If you are using BIG-IP, install f5-iapplx-msda-nacos RPM package using iApps->Package Management LX->Import screen. To create an application, use iApps-> Templates LX -> Application Services -> Applications LX -> Create screen. Default IApp LX UI will be rendered based on the input properties specified in basic pool IAppLX.

## Using IAppLX from REST API to configure BIG-IP

Using the REST API to work with BIG-IP with f5-iapplx-msda-nacos IAppLX package installed. 

Create an Application LX block with all inputProperties as shown below.
Save the JSON to block.json and use it in the curl call. Refer to the clouddoc link for more detail: https://clouddocs.f5.com/products/iapp/iapp-lx/tmos-14_0/iapplx_ops_tutorials/creating_iappslx_with_rest.html .

```json
{
  "name": "msdanacos",
  "inputProperties": [
    {
      "id": "nacosEndpoint",
      "type": "STRING",
      "value": "http://1.1.1.1:8848",
      "metaData": {
        "description": "Nacos endpoint list, eg. http://1.1.1.1:8848 or https://1.1.1.2:8848",
        "displayName": "Nacos endpoints",
        "isRequired": true
      }
    },
    {
      "id": "nacosUserName",
      "type": "STRING",
      "value": "nacos",
      "metaData": {
        "description": "Nacos username",
        "displayName": "Nacos Username",
        "isRequired": false
      }
    },
    {
      "id": "nacosPassword",
      "type": "STRING",
      "value": "nacos",
      "metaData": {
        "description": "Nacos password",
        "displayName": "Nacos Password",
        "isRequired": false
      }
    },
    {
      "id": "namespaceId",
      "type": "STRING",
      "value": "",
      "metaData": {
        "description": "Namespce ID of the service, NOT the name, eg 41e92cf7-f178-46fa-8038-49e2e9f0d754",
        "displayName": "NamespaceId in registry",
        "isRequired": false
      }
    },
    {
      "id": "serviceName",
      "type": "STRING",
      "value": "msda.nacos.com",
      "metaData": {
        "description": "Service name to be exposed, can include the namespaceId, eg nacosmsda.naf5demo.com&namespaceId=41e92cf7-f178-46fa-8038-49e2e9f0d754",
        "displayName": "Service Name in registry",
        "isRequired": true
      }
    },
    {
      "id": "poolName",
      "type": "STRING",
      "value": "/Common/nacosSamplePool",
      "metaData": {
        "description": "Pool Name to be created",
        "displayName": "BIG-IP Pool Name",
        "isRequired": true
      }
    },
    {
      "id": "poolType",
      "type": "STRING",
      "value": "round-robin",
      "metaData": {
        "description": "load-balancing-mode",
        "displayName": "Load Balancing Mode",
        "isRequired": true,
        "uiType": "dropdown",
        "uiHints": {
          "list": {
            "dataList": [
              "round-robin",
              "least-connections-member",
              "least-connections-node"
            ]
          }
        }
      }
    },
    {
      "id": "healthMonitor",
      "type": "STRING",
      "value": "none",
      "metaData": {
        "description": "Health Monitor",
        "displayName": "Health Monitor",
        "isRequired": true,
        "uiType": "dropdown",
        "uiHints": {
          "list": {
            "dataList": ["tcp", "udp", "http", "none"]
          }
        }
      }
    }
  ],
  "dataProperties": [
    {
      "id": "pollInterval",
      "type": "NUMBER",
      "value": 30,
      "metaData": {
        "description": "Interval of polling from BIG-IP to registry, 30s by default.",
        "displayName": "Polling Invertal",
        "isRequired": false
      }
    }
  ],
  "configurationProcessorReference": {
    "link": "https://localhost/mgmt/shared/iapp/processors/msdanacosConfig"
  },
  "configProcessorTimeoutSeconds": 30,
  "statsProcessorTimeoutSeconds": 15,
  "configProcessorAffinity": {
    "processorPolicy": "LOAD_BALANCED",
    "affinityProcessorReference": {
      "link": "https://localhost/mgmt/shared/iapp/processors/affinity/load-balanced"
    }
  },
  "auditProcessorReference": {
    "link": "https://localhost/mgmt/shared/iapp/processors/msdanacosEnforceConfiguredAudit"
  },
  "audit": {
    "intervalSeconds": 60,
    "policy": "ENFORCE_CONFIGURED"
  },
  "state": "TEMPLATE"
}
```

Post the block through REST API using curl. 
```bash
curl -sk -X POST -d @block.json https://bigip_mgmt_ip:8443/mgmt/shared/iapp/blocks
```
