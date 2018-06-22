# TripletexSession
C# library to create orders and send invoices from Tripletex

See documentation for the API over at https://tripletex.no/v2-docs/.

## Sample usage

```csharp
using (var session = new TripletexSession())
{
    try
    {
        var consumerKey = CloudConfigurationManager.GetSetting("TripletexConsumerKey");
        var employeeKey = CloudConfigurationManager.GetSetting("TripletexEmployeeKey");
        var productNumber = CloudConfigurationManager.GetSetting("TripletexProductNumber");
        if (await session.CreateSessionToken(consumerKey, employeeKey))
        {
            var customer = await session.GetCustomerFromCustomerNo(customerNo, "id", "name");
            var prodNo = productNumber;
            var product = await session.GetProductFromNumber(prodNo, "id", "name", "vatType(id)",
                "priceExcludingVatCurrency");

            if (customer != null && product != null)
            {
                var order = new JObject
                {
                    {"deliveryDate", lastDayOfMonth},
                    {"orderDate", lastDayOfMonth},
                    {"customer", customer}
                };
                var rsp = await session.CreateOrder(order);
                var orderLine = new JObject
                {
                    {"order", rsp},
                    {"product", product},
                    {"description", $"{firstDayOfMonth:yyyy-MM-dd} - {lastDayOfMonth:yyyy-MM-dd}"},
                    {"count", count},
                    {"vatType", product["vatType"]},
                    {"unitPriceExcludingVatCurrency", product["priceExcludingVatCurrency"]}
                };
                _ = await session.CreateOrderLine(orderLine);
                try
                {
                    var invoice = await session.CreateInvoice(order["id"].Value<int>(), lastDayOfMonth);
                        log.Info($"Created and sent invoice {invoice["invoiceNumber"]}");
                    return invoice;
                }
                catch (Exception e)
                {
                    log.Error($"Could not create invoice of order {order["orderNumber"]}");
                }
            }
        }
    }
    finally
    {
        await session.DestroySessionToken();
    }
}
```
