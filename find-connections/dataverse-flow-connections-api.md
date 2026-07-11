# Dataverse Web API — Flows & Connections Reference

Reference for a future HTML tool that streamlines finding flows and mapping them to
connections. All calls are `GET`, same-origin, relying on the authenticated browser session.

## Base URL

```
https://contoso.api.crm.dynamics.com/api/data/v9.2
```

### Recommended headers

```
Accept: application/json
OData-MaxVersion: 4.0
OData-Version: 4.0
Prefer: odata.include-annotations="*"   // optional, for formatted lookup values
```

### Known values (this org)

| Item | Value |
|------|-------|
| Current user `UserId` (owner GUID) | `00000000-0000-0000-0000-000000000001` |
| BusinessUnitId | `00000000-0000-0000-0000-000000000002` |
| OrganizationId | `00000000-0000-0000-0000-000000000003` |

---

## Endpoints

### 1. Identify the current user

```
GET /WhoAmI
```

Unbound function. Returns `UserId`, `BusinessUnitId`, `OrganizationId`.
Use `UserId` as the owner GUID in the filters below.

### 2. List cloud flows

```
GET /workflows
  ?$select=name,category,statecode,createdon,modifiedon,workflowidunique
  &$filter=category eq 5 and _ownerid_value eq {UserId}
  &$orderby=modifiedon desc
```

- `category eq 5` -> modern/cloud flows (0 = classic workflow).
- `_ownerid_value eq {guid}` -> lookup filters require the `_..._value` form.
- `statecode`: `0` = Draft/Off, `1` = Activated/On.
- Drop the `_ownerid_value` clause to list the whole environment.

### 3. List connection references

```
GET /connectionreferences
  ?$select=connectionreferencedisplayname,connectionreferencelogicalname,connectorid,connectionid,statecode,createdon,modifiedon
  &$filter=_ownerid_value eq {UserId}
  &$orderby=modifiedon desc
```

Key columns:
- `connectionreferencelogicalname` — the join key into flow definitions.
- `connectorid` — e.g. `/providers/Microsoft.PowerApps/apis/shared_sharepointonline`.
- `connectionid` — the underlying connection instance.

### 4. Flow definitions (for the flow -> connection mapping)

```
GET /workflows
  ?$select=name,clientdata,statecode
  &$filter=category eq 5 and _ownerid_value eq {UserId}
```

`clientdata` is a JSON **string**. Parse it, then read:

```
properties.connectionReferences[*].connection.connectionReferenceLogicalName  // -> join to #3
properties.connectionReferences[*].api.name                                   // -> connector, e.g. shared_sharepointonline
properties.connectionReferences[*].connection.id                              // -> raw connection (non-solution flows)
```

---

## Tool-builder notes

- **Two tables only**: `workflows` and `connectionreferences`. `WhoAmI` seeds the owner filter.
- **The join is client-side**: match `clientdata -> connectionReferenceLogicalName` against
  `connectionreferences.connectionreferencelogicalname`. There is no server-side relationship to `$expand`.
- **Watch response size**: `clientdata` is large — select it only when building the mapping, not for list views.
- **Paging**: Dataverse caps at 5000 rows and returns `@odata.nextLink`. Add header
  `Prefer: odata.maxpagesize=N` and follow `nextLink` if you exceed it.
- **Orphan detection** (natural tool feature): connection references whose logical name appears
  in *zero* flow definitions are unused and safe cleanup candidates.

### Reference extraction snippet (browser, same-origin session)

```js
const base = "https://contoso.api.crm.dynamics.com/api/data/v9.2";
const uid  = "00000000-0000-0000-0000-000000000001";

const resp = await fetch(
  `${base}/workflows?$select=name,clientdata,statecode` +
  `&$filter=category eq 5 and _ownerid_value eq ${uid}`,
  { headers: { Accept: "application/json" } }
);
const { value } = await resp.json();

function extract(cd) {
  const refs = new Set(), connectors = new Set();
  if (!cd) return { refs: [], connectors: [] };
  let obj; try { obj = JSON.parse(cd); } catch { return { refs: [], connectors: [] }; }
  const cr = obj?.properties?.connectionReferences ?? obj?.connectionReferences ?? {};
  for (const v of Object.values(cr)) {
    if (v?.connection?.connectionReferenceLogicalName) refs.add(v.connection.connectionReferenceLogicalName);
    if (v?.api?.name) connectors.add(v.api.name.split("/").pop());
  }
  return { refs: [...refs], connectors: [...connectors] };
}

const map = value.map(w => ({ name: w.name, state: w.statecode, ...extract(w.clientdata) }));
```

---

## Reference data: connection reference display names (this org)

| Logical name | Connector | Display name |
|--------------|-----------|--------------|
| `new_sharedonedriveforbusiness_dc7bf` | shared_onedriveforbusiness | OneDrive for Business |
| `new_sharedwebcontents_f9b80` | shared_webcontents | HTTP with Entra ID (preauth) |
| `new_sharedsharepointonline_c2e01` | shared_sharepointonline | SharePoint |
| `new_sharedsharepointonline_b03a5` | shared_sharepointonline | SharePoint |
| `new_sharedsharepointonline_3c541` | shared_sharepointonline | SharePoint |
| `new_sharedoffice365_77dfb` | shared_office365 | Office 365 Outlook |
| `new_sharedoutlook_f9c5a` | shared_outlook | Outlook.com |
