# Update oauth2permissiongrant

Update the properties of oauth2permissiongrant object.
### Prerequisites
The following **scopes** are required to execute this API: 
### HTTP request
<!-- { "blockType": "ignored" } -->
```http
PATCH /oauth2PermissionGrants/<id>
PATCH /users/<id | userPrincipalName>/oauth2PermissionGrants/<id>
PATCH /drive/root/createdByUser/oauth2PermissionGrants/<id>
```
### Request headers
| Name       | Type | Description|
|:-----------|:------|:----------|
| Authorization  | string  | Bearer <token>. Required. |

### Request body
In the request body, supply the values for relevant fields that should be updated. Existing properties that are not included in the request body will maintain their previous values or be recalculated based on changes to other property values. For best performance you shouldn't include existing values that haven't changed.

| Property	   | Type	|Description|
|:---------------|:--------|:----------|
|clientId|String||
|consentType|String||
|expiryTime|DateTimeOffset||
|principalId|String||
|resourceId|String||
|scope|String||
|startTime|DateTimeOffset||

### Response
If successful, this method returns a `200 OK` response code and updated [oAuth2PermissionGrant](../resources/oauth2permissiongrant.md) object in the response body.
### Example
##### Request
Here is an example of the request.
<!-- {
  "blockType": "request",
  "name": "update_oauth2permissiongrant"
}-->
```http
PATCH https://graph.microsoft.com/beta/oauth2PermissionGrants/<id>
Content-type: application/json
Content-length: 180

{
  "clientId": "clientId-value",
  "consentType": "consentType-value",
  "expiryTime": "datetime-value",
  "principalId": "principalId-value",
  "resourceId": "resourceId-value"
}
```
##### Response
Here is an example of the response. Note: The response object shown here may be truncated for brevity. All of the properties will be returned from an actual call.
<!-- {
  "blockType": "response",
  "truncated": true,
  "@odata.type": "microsoft.graph.oauth2permissiongrant"
} -->
```http
Content-type: application/json
Content-length: 200

{
  "clientId": "clientId-value",
  "consentType": "consentType-value",
  "expiryTime": "datetime-value",
  "id": "id-value",
  "principalId": "principalId-value",
  "resourceId": "resourceId-value"
}
```

<!-- uuid: 8fcb5dbc-d5aa-4681-8e31-b001d5168d79
2015-10-25 14:57:30 UTC -->
<!-- {
  "type": "#page.annotation",
  "description": "Update oauth2permissiongrant",
  "keywords": "",
  "section": "documentation",
  "tocPath": ""
}-->