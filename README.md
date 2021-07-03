### Todo

- [ ] Create typescript library `@saxolab/ispec`

- [ ] Create diff library

  - [ ] Change return type to

    ```yaml
    {
      path : 'Snapshots.Rows[3].Display.Currency', 
      cloud: undefined, 
      legacy: 'GB'
    }
    ```

  - [ ] Create diff of arrays, and nested objects

  - [ ] How to handle when arrays have a mismatch of elements

- [ ] Move config to env variables and cli params

  

This is a tool for creating acceptance reports for new cloud services.



### Features

- Auto generate tokens 
- Publish reports to wiki
- Generate markdown by running on local
- use as cli tool or library



**How to use**

- Declarative Style : 
  - Create a simple javascript file, set env variables and run using cli
- Library Style : 
  - Create purly custom logic by using individual components like 
    - SaxoApiClient
    - SaxoTokenGenerator
    - Report API
    - WikiPublisher
    - Diff Calculator



### Declarative Style

Install project globally

```
npm i -g saxolab/ispec
```



Create script `exchange-rate.ispec.js`

```javascript
const currencyPairsToTest = [
  "USDINR",
  "INRDKK",
  "XYZABC",
  "usdInR",
  "usdinr"
];

module.exports =  async({client, wiki, reports, diff, config, files}) => {

  await Promise.all(currencyPairsToTest.map(async (currencyPair) => {
    const report = reports.create();
    report.AddHeading(currencyPair);

    const cloudResponse  = client.get(`cloud-url/prices/v1/exchangerates?pair=${currencyPair}`);

    const onPremResponse  = client.get(`on-prem-url/prices/v1/exchangerates?pair=${currencyPair}`);

    const diff = diff.compare({legacy : onPremResponse.data, cloud: cloudResponse.data});

    // Create heading for table in report
    const table = diff.map(d => {
      return {
        'Field Name': d.fieldName,
        'Value on cloud': d.cloudValue,
        'Value on premises': d.legacyValue,
      };
    });

    report.addTable(table);

    // Publish report on wiki
    wiki.publish(currencyPair, config.exchnageRatePage, report.getHtml());

    files.save(`${currencyPair-cloud.json}`, cloudResponse.data);
    files.save(`${currencyPair-onprem.json}`, onPremResponse.data);

    // Save markdown to view on local
    files.save(`${currencyPair-report.md}`, report.getMarkdDown());
  }));
}
```



Running spec

```bash
// Set WIKI_TOKEN env variable
ispec exchange-rate.ispec.js env=tst211 publish=true
```



### Library Style

```javascript
import ispec from 'saxolab/ispec';

const {client, wiki, reports, diff, config, files} = ispec.initialize();

const currencyPairsToTest = [
  "USDINR",
  "INRDKK",
  "XYZABC",
  "usdInR",
  "usdinr"
];

(async() => {

    await Promise.all(currencyPairsToTest.map(async (currencyPair) => {
      const report = reports.create();
      report.AddHeading(currencyPair);

      const cloudResponse  = client.get(`cloud-url/prices/v1/exchangerates?pair=${currencyPair}`);

      const onPremResponse  = client.get(`on-prem-url/prices/v1/exchangerates?pair=${currencyPair}`);

      const diff = diff.compare({legacy : onPremResponse.data, cloud: cloudResponse.data});

      // Create heading for table in report
      const table = diff.map(d => {
        return {
          'Field Name': d.fieldName,
          'Value on cloud': d.cloudValue,
          'Value on premises': d.legacyValue,
        };
      });

      report.addTable(table);

      // Publish report on wiki
      wiki.publish(currencyPair, config.exchnageRatePage, report.getHtml());

      files.save(`${currencyPair-cloud.json}`, cloudResponse.data);
      files.save(`${currencyPair-onprem.json}`, onPremResponse.data);

      // Save markdown to view on local
      files.save(`${currencyPair-report.md}`, report.getMarkdDown());
    }));

  }
)();
```



### CLI options

| Parameter | Required/Optional                               |                                                              |
| --------- | ----------------------------------------------- | ------------------------------------------------------------ |
| `env`     | required if `API_TOKEN` env variable is not set | Used to get token. Valide values  can be `env=red`, `env=blue`, `env=tst231`,  or any DTE instance supported by assumption service |
| `publish` | optional (default is false)                     | If not set to true, wiki will not be updated                 |
|           |                                                 |                                                              |



### Environment variables

| Env name     | Required/Optional                   |      |
| ------------ | ----------------------------------- | ---- |
| `WIKI_TOKEN` | required if cli param`publish=true` |      |
| `API_TOKEN`  | required if `env` not provided      |      |
| `WIKI_URL`   | optional (defaults to https://wiki) |      |





```
WIKI_TOKEN
```

