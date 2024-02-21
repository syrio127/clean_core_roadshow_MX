# CAP Backend Development - BTP Clean Core Roadshow

## Schema.cds snippet

```js
entity Contracts : managed {
	key ID: String;
	description: String;
	beginDate: Date;
	endDate: Date;
	totalMonthValue: Integer;
	totalMonthCurrency: String;
	// BusinessPartner: Association to one BusinessPartner;
	contractItems: Composition of many ContractItems on contractItems.contract = $self;
}

@cds.autoexpose
	entity ContractItems : cuid {
	key contract: Association to one Contracts;
	beginDate: Date;
	endDate: Date;
	price: Integer;
	priceCurrency: String;
	// tool: Association to one Tools;
	toolName: String
}

```


## admin-service.cds
```js
using { p2c.cap.contracts as my } from '../db/schema';

service AdminService @(path: '/admin'){

    entity Contracts as projection on my.Contracts;
    entity ContractItems as projection on my.ContractItems;

}
```
## contracts-service.cds
```js
using { p2c.cap.contracts as my} from '../db/schema';

service  ContractsService @(path:'/contracts'){

    entity Contracts as select from my.Contracts {
        ID, description
    };

    @readonly entity ContractItems as projection on my.ContractItems;


}
```
## p2c.cap.contracts-Contracts.csv
```csv
ID,description,beginDate,endDate,totalMonthValue,totalMonthCurrency
CN-100,CCL Mining CO - Perforation 2023,2023-03-01,2023-10-25,0,USD
CN-105,Pacific Transportation INC. South Africa Mining,2022-02-02,2028-04-02,0,USD
CN-106,Atlantic Transportation INC. - Cooper Mine-Port Movement,2023-02-02,2026-05-18,0,USD
CN-108,Pure Gold Company - Explosives and Perforation, 2023-02-02,2025-12-02,0,USD
CN-109,Rusell & CO Diamond Mining - Australian Full Operation Contract,2023-03-03,2030-10-18,0,USD
```

## p2c.cap.contracts-ContractItems.csv
```csv
ID,contract_ID,beginDate,endDate,price,priceCurrency
02d29d9d-3538-4140-8381-67ee542928db,CN-100,2023-03-01,2023-03-30,5700,USD
0b9e270f-0d11-4dd1-bb52-976812bcd32f,CN-100,2023-03-01,2023-03-10,180000,USD
63f57654-ee43-4f05-8fa2-7fd3e240517f,CN-105,2022-02-02,2022-02-02,190,USD
a324e459-a773-4616-8f48-dc60a3ab890f,CN-106,2023-02-02,2023-02-02,1400,USD
ac184d7a-b485-493e-be77-9d34c2d76407,CN-108,2023-02-02,2023-02-02,17000,USD
cef9a93e-2f97-47ec-8256-7ae3ef7b7704,CN-109,2023-03-03,2023-03-03,17000,USD
```

## external-test.cds
```js
using { p2c.cap.contracts as my } from '../db/schema';


service DemoService @(path: '/demosrv'){

    //**Business Partner integration from S4HANA */

    @readonly
    // entity BusinessPartners as projection on my.BusinessPartners;
    entity Contracts as projection on my.Contracts;
    entity ContractItems as projection on my.ContractItems;
}
```

## API_BUSINESS_PARTNER-A_BusinessPartner.csv
```csv
BusinessPartner;BusinessPartnerFullName;BusinessPartnerIsBlocked
1000038;Williams Electric Drives;false
1000040;Smith Batteries Ltd;false
1000042;Johnson Automotive Supplies;true
```

## Add to schema.cds
```js
using { API_BUSINESS_PARTNER as bpar } from '../srv/external/API_BUSINESS_PARTNER.csn';

entity BusinessPartners as projection on bpar.A_BusinessPartner {
    key BusinessPartner,
    BusinessPartnerFullName,
    BusinessPartnerIsBlocked 
}
```
_________________________________________________________________

## Replace CSV files for contracts

### Contracts
```csv
ID,description,beginDate,endDate,totalMonthValue,totalMonthCurrency,businessPartner_BusinessPartner
CN-100,CCL Mining CO - Perforation 2023,2023-03-01,2023-10-25,0,USD,203
CN-105,Pacific Transportation INC. South Africa Mining,2022-02-02,2028-04-02,0,USD,1018
CN-106,Atlantic Transportation INC. - Cooper Mine-Port Movement,2023-02-02,2026-05-18,0,USD,1710
CN-108,Pure Gold Company - Explosives and Perforation, 2023-02-02,2025-12-02,0,USD,1710
CN-109,Rusell & CO Diamond Mining - Australian Full Operation Contract,2023-03-03,2030-10-18,0,USD,1018

```

## Install additional packages (as an administrator)
```npm install @sap-cloud-sdk/http-client @sap-cloud-sdk/util ```


## Add to package.json in API_BUSINESS_PARTNER definition
```json
"[sandbox]":{
          "credentials": {
            "url": "https://sandbox.api.sap.com/s4hanacloud/sap/opu/odata/sap/API_BUSINESS_PARTNER"
          }
        }
```

## Create external-test.js (implementation service)
```js
const cds = require('@sap/cds')

module.exports = async (srv) => {

//* Local Service Entities
const {Contracts, BusinessPartners, ContractItems } = srv.entities;

// //* Tools API connection
// const toolsAPI = await cds.connect.to("toolsManager")

//*Business Partner Connect
const bupa = await cds.connect.to('API_BUSINESS_PARTNER')
const {A_BusinessPartner} = bupa.entities;


//* Entity BusinessPartners READ -- oData
srv.on('READ', 'BusinessPartners', async (req) => {
	return await bupa.transaction(req).send({
		query: req.query,
		headers: {
		    apikey: "lSnMaBSXTGh7YX1aLfAyN08AdDkVRsfz"

		}
	})
})
}
```
## Run with sandbox profile
`cds watch --profile sandbox`


## replace cds section in package.json
```json
  "cds": {
    "requires": {
      "API_BUSINESS_PARTNER": {
        "kind": "odata-v2",
        "model": "srv/external/API_BUSINESS_PARTNER",
        "[sandbox]":{
          "credentials": {
            "url": "https://sandbox.api.sap.com/s4hanacloud/sap/opu/odata/sap/API_BUSINESS_PARTNER"
          }
        }
      },
      "toolsManager": {
        "kind": "rest",
        "credentials": {
          "url": "http://localhost:8080",
          "requestTimeout": 3000
        },
        "[sandbox]": {
          "credentials": {
            "url": "https://p2c-poc-rest-srv-ts-production.up.railway.app/"
          }
        },
        "[production]": {
          "credentials": {
            "destination": "P2C-ToolManager"
          }
        }
    }
  }
}
}
```


## Add functions to external-test.cds
```cds
 //* Non SAP Source Access Functions    

    function getTools() returns {
        totalTools: Integer;
        activeTools: Integer;
        msg: String;

        toolsList: array of {
            _id: String;
            toolName: String;
            toolDescription: String;
            toolStatus: Boolean;
            toolDailyPrice: Integer;
            toolCurrency: String;
            //__v: Integer;
            toolAvailable: Boolean;
            toolCategory: String
        }
    };

    function getToolById(id: String) returns {
        msg: String;
        tool: {
            _id: String;
            toolName: String;
            toolDescription: String;
            toolStatus: Boolean;
            toolDailyPrice: Integer;
            toolCurrency: String;
            //__v: Integer;
            toolAvailable: Boolean;
            toolCategory: String
        }
    };

function changeToolStatus(id: String) returns {
	msg: String;
};

```

## Add implementation to external-test.js
```js
//* REST API Function Calls for Tools */
//**Function Read Tools -- NO oData

srv.on('getTools',async (req) => {

	const toolsAPI = await cds.connect.to("toolsManager")
	return await toolsAPI.send({query: 'GET /api/tools', headers:{
		userName: "p2c-comm-user",
		APIKey: "K306GLEZ1TPZZU4CIVGTQFQHXZ2920"
	}})
})

//* Function Read Tool By ID -- No oData
//* URL de Consulta: http://localhost:4004/demosrv/getToolById(id='64da6b6e55b27b4c05d262c1')

srv.on('getToolById',async (req) => {

	const { id } = req.data
	const toolsAPI = await cds.connect.to("toolsManager")
	// return toolsAPI.tx(req).get(`/api/tools/${id}`)
	return await toolsAPI.send({query:`GET /api/tools/${id}`, headers:{
		userName: "p2c-comm-user",
		APIKey: "K306GLEZ1TPZZU4CIVGTQFQHXZ2920"
	}})
})

//* Function to change boolean status of a tool (available), to be called on
//* contract item creation

srv.on('changeToolStatus', async (req) => {

	const { id } = req.data
	const toolsAPI = await cds.connect.to("toolsManager")
	return await toolsAPI.send({query:`PATCH /api/tools/${id}`, headers: {
		userName: "p2c-comm-user",
		APIKey: "K306GLEZ1TPZZU4CIVGTQFQHXZ2920"
	}})
})

```

