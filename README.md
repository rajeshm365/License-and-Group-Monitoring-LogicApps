# License & Group Threshold Monitoring (Logic Apps)

Two Azure Logic Apps using **Managed Identity** + **Microsoft Graph** to keep an eye on:
- **License usage** (e.g., E3/E5): alerts when % remaining < threshold
- **Monitored group membership capacity**: alerts when % capacity left < threshold

## Files
- `src/LogicApp-LicenseUsage.json` – daily check of `subscribedSkus`, filters SKUs, computes % left, posts Teams table if below threshold
- `src/LogicApp-GroupThresholds.json` – daily group scan (Graph), computes capacity left vs 200k, posts Teams table if below threshold

## Setup
1. **Create Logic Apps** (Consumption or Standard) and enable **System-Assigned Managed Identity**.
2. **Grant Managed Identity** Graph perms:
   - License app: `Directory.Read.All` (to read `subscribedSkus`)
   - Group app: `Group.Read.All`
   - (Admin consent required)
3. **Import JSON** in each Logic App (Code view) and set parameters:
   - `teamId`, `channelId` (Teams)
   - `licenseSkuFilter`, `thresholdPercent`
   - `groupIds`, `groupCapacity`, `thresholdLeft`
4. **Add Teams connection** in Logic App designer; it will populate `$connections`.
5. **Schedule** – recurrences are set to daily by default.

## Output
Teams **Adaptive Card** table with:
- **License**: Name | Total | Available | Used | %Remaining
- **Groups**: Group Name | Total | Available | Used | %Left

## 🏗️ Architecture

### High level
```mermaid
flowchart TD
    A[Logic App license] --> LGET[Graph get subscribedSkus]
    LGET --> LPROC[Filter SKUs and compute percent remaining]
    LPROC --> LCHECK{Any below threshold}
    LCHECK -- No --> LEND[No post]
    LCHECK -- Yes --> LPOST[Post adaptive card to Teams]

    B[Logic App groups] --> GLOOP[Iterate monitored group ids]
    GLOOP --> GGET[Graph get group details and member count]
    GGET --> GPROC[Compute capacity left and percent left]
    GPROC --> GCHECK{Any below threshold}
    GCHECK -- No --> GEND[No post]
    GCHECK -- Yes --> GPOST[Post adaptive card to Teams]

## Notes
- Lists (`licenseSkuFilter`, `groupIds`) are **expandable**.
- Adaptive Card 1.6 **Table** used; keep payload sizes sensible.
- For large tenants, consider paging or batching.
