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

