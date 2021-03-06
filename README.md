# Azure Functions v2 with .NET Core - Serverless in Insurance world

This example shows how you can use the serverless architecture in insurance billing system.

<p align="center">
    <img alt="Architecture" src="https://raw.githubusercontent.com/asc-lab/dotnetcore-azure-functions/master/readme-images/azure-functions-architecture.png" />
</p>

1. User uploads CSV file (with name structure ```CLIENTCODE_YEAR_MONTH_activeList.txt.```) with Beneficiaries (the sample file is located in the ```data-examples``` folder) to a specific data storage - ```active-lists``` Azure Blob Container.
2. The above action triggers a function (```GenerateBillingItemsFunc```) that is responsible for:
    * generating billing items (using prices from an external database - CosmosDB ```crm``` database, ```prices``` collection) and saving them in the table ```billingItems```;
    * sending message about the need to create a new invoice to ```invoice-generation-request```;

3. When a new message appears on the queue ```invoice-generation-request```, next function is triggered (```GenerateInvoiceFunc```). This function creates domain object ```Invoice``` and save this object in database (CosmosDB ```crm``` database, ```invoices``` collection) and send message to queues: ```invoice-print-request``` and ```invoice-notification-request```.
4. When a new message appears on the queue ```invoice-print-request```, function ```PrintInvoiceFunc``` is triggered. This function uses external engine to PDF generation - JsReport and saves PDF file in BLOB storage.
5. When a new message appears on the queue ```invoice-notification-request```, function ```NotifyInvoiceFunc``` is triggered. This function uses two external systems - SendGrid to Email sending and Twilio to SMS sending.

## Prerequisites

1. Install and run [Microsoft Azure Storage Emulator](https://docs.microsoft.com/en-us/azure/storage/common/storage-use-emulator).
2. Install and run [CosmosDB](https://docs.microsoft.com/en-us/azure/cosmos-db/local-emulator).
3. Create blob Container ```active-lists```.
4. Upload  ```ASC_2018_02_activeLists.txt``` file from ```data-examples``` folder to ```active-lists``` blob.
5. Create CosmosDB database ```crm``` and collections: ```prices```,  ```invoices```.
6. Run project ```PriceDbInitializator``` to init collection with prices.
7. Run JsReport ```docker run -p 5488:5488 jsreport/jsreport```. Check JsReport Studio on ```localhost:5488```.
8. Add JsReport url as ```JsReportUrl``` to ```local.settings.json``` in ```PrintInvoiceFunc``` project.
9. Create an account in [Twilio](https://www.twilio.com/) and add properties ```TwilioAccountSid``` ```TwilioAuthToken``` to ```local.settings.json```
10. Create an account in [SendGrid](https://sendgrid.com/) and add property ```SendGridApiKey``` to ```local.settings.json``` in ```NotifyInvoiceFunc``` project.
11. Add CosmosDB properties ```PriceDbUrl``` and ```PriceDbAuthKey``` to ```local.appsettings.json``` in ```PriceDbInitializator```.
12. Add CosmosDB connection string as ```cosmosDb``` to ```local.settings.json``` in ```GenerateInvoiceFunc``` project.
13. Add JsReport template with name ```INVOICE``` and content:

```html
<h1>Invoice {{invoiceNumber }}</h1>
<h3>Customer: {{ customer }}</h3>
<h3>Address: 00-101 Warszawa, Chłodna 21</h3>
<h3>Description: {{ description }}</h3>

<h3>Details:</h3>
<table width="90%" border="1" bgcolor="#C0C0C0" align="center">
    <tr>
        <th>Item</th>
        <th>Price</th>
    </tr>

    {{#each lines}}
    <tr>
        <td>{{ itemName }}</td>
        <td align="right">{{ cost }}</td>
    </tr>
    {{/each}}

    <tr>
        <td>
            <strong>Total</strong>
        </td>
        <td align="right">
            <strong>{{ totalCost }}</strong>
        </td>
    </tr>
</table>
```

Example JSON for INVOICE template:

```json
{
  "customer": "ASC",
  "invoiceNumber": "ASC/10/2018",
  "description": "Invoice for insurance policies for 10/2018",
  "lines": [
    {
      "itemName": "Policy A",
      "cost": 2140.0
    },
    {
      "itemName": "Policy B",
      "cost": 1360.0
    }
  ],
  "totalCost": 3500.0
}
```

## Tips & Tricks

1. CSV file is working for client code ```ASC``` (filename: ```ASC_2018_12_activeList.txt```). If you want run functions for another client code, you must simulate prices in database. Check project ```PriceDbInitializator```, file ```Program.cs```, method ```AddDoc```.

2. Remember that you must use **Twilio Test Credentials**.

<p align="center">
    <img alt="Twilio Test Credentials" src="https://raw.githubusercontent.com/asc-lab/dotnetcore-azure-functions/master/readme-images/twilio_test_credentials.png" />
</p>

3. **Microsoft Azure Storage Emulator** with all created storages:

<p align="center">
    <img alt="Microsoft Azure Storage Emulator" src="https://raw.githubusercontent.com/asc-lab/dotnetcore-azure-functions/master/readme-images/azure_storage_emulator.png" />
</p>
