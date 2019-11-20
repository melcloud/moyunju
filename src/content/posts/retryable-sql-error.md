---
title: "Retryable SQL Error"
date: 2019-11-21T08:37:25+11:00
draft: false
toc: false
images:
tags:
  - Azure
  - Azure SQL
  - SQL
  - dotnet core
  - resilient system
---

As indicate in post [Azure SQL transient errors](https://hackernoon.com/azure-sql-transient-errors-7625ad6e0a06) by [Mattias Karlsson](https://hackernoon.com/@devlead?source=post_header_lockup), the following SQL errors are safe to retry:

| Error                                                  | Error Number |
|--------------------------------------------------------|--------------|
| SqlErrorInstanceDoesNotSupportEncryption               | 20           |
| SqlErrorConnectedButLoginFailed                        | 64           |
| SqlErrorUnableToEstablishConnection                    | 233          |
| SqlErrorTransportLevelErrorReceivingResult             | 10053        |
| SqlErrorTransportLevelErrorWhenSendingRequestToServer  | 10054        |
| SqlErrorNetworkRelatedErrorDuringConnect               | 10060        |
| SqlErrorDatabaseLimitReached                           | 10928        |
| SqlErrorResourceLimitReached                           | 10929        |
| SqlErrorServiceErrorEncountered                        | 40197        |
| SqlErrorServiceBusy                                    | 40501        |
| SqlErrorServiceRequestProcessFail                      | 40540        |
| SqlErrorServiceExperiencingAProblem                    | 40545        |
| SqlErrorDatabaseUnavailable                            | 40613        |
| SqlErrorOperationInProgress                            | 40627        |


I am not quite sure if I want to retry *SqlErrorConnectedButLoginFailed* or *SqlErrorInstanceDoesNotSupportEncryption* as these errors sound determined to me.
But I guess it is based on the context. For a complete list of error codes, refer to Microsoft documentation [here](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-develop-error-messages). The [Connectivity guidance](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-connectivity-issues) is worth reading as well.

