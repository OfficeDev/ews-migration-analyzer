# EWS to Graph API Operation Mapping Reference

See the [EWS to Graph API Mappings](https://aka.ms/ews2graphMap) for the full mapping.

## Operation Mapping Table

| EWS Operation | Graph API Equivalent | Documentation |
|---|---|---|
| `ExchangeService.FindItems(Inbox, ItemView)` | `GET /me/messages?$top=N&$orderby=receivedDateTime desc` | [List messages](https://learn.microsoft.com/en-us/graph/api/user-list-messages) |
| `EmailMessage.Bind(service, id, PropertySet)` | `GET /me/messages/{id}?$select=...` | [Get message](https://learn.microsoft.com/en-us/graph/api/message-get) |
| `EmailMessage.CreateReply(false)` + `reply.SendAndSaveCopy()` | `POST /me/messages/{id}/reply` | [Reply to message](https://learn.microsoft.com/en-us/graph/api/message-reply) |
| `EmailMessage.Send()` / `SendAndSaveCopy()` | `POST /me/sendMail` | [Send mail](https://learn.microsoft.com/en-us/graph/api/user-sendmail) |
| `ExchangeService.FindFolders()` | `GET /me/mailFolders` | [List mailFolders](https://learn.microsoft.com/en-us/graph/api/user-list-mailfolders) |
| `Folder.Bind(service, folderId)` | `GET /me/mailFolders/{id}` | [Get mailFolder](https://learn.microsoft.com/en-us/graph/api/mailfolder-get) |
| `OAuthCredentials` setup | Bearer token via MSAL / Microsoft.Identity.Web | [Auth overview](https://learn.microsoft.com/en-us/graph/auth/) |

## Handling Parity Gaps

If the discovery report (Skill 00) identified EWS operations without Graph equivalents (EWS003):

1. **Check the current roadmap**: <https://learn.microsoft.com/en-us/exchange/clients-and-mobile-in-exchange-online/deprecation-of-ews-exchange-online#roadmap-for-parity-gaps>
2. **Current known gaps** (as of 2025):
   - Import/export of public folders
   - Import/export of Microsoft 365 Groups
   - Archive operations
   - Event delta for recurring events
   - Sticky Notes CRUD
3. **Options**:
   - Wait for Graph parity (check ETAs on roadmap)
   - Implement workaround (alternative Graph APIs or Power Platform)
   - Keep EWS for specific gap operations while migrating everything else
   - Escalate to Microsoft via support or feedback channels
