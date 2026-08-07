# Set Renewal Uplift (CPI Price Index) to All Quote Lines

A Salesforce **Screen Flow** that applies a **Price Revision Policy** (CPI / price
index uplift) to **all quote line items at once**, instead of setting the policy
line by line.

> **Target org:** an existing **sandbox / Developer Edition / dev** org.
> **Never deploy to production** from this flow.

---

## What's included

```
Set_Renewal_Price_Uplift_Policy.flow-meta.xml   # a single Screen Flow, at the repo ROOT
README.md
*.jpg                                            # prerequisite + flow screenshots
```

Deploy mechanism: **single Flow metadata file** — there is **no
`sfdx-project.json`** in this repo, so wrap it in a minimal project (below) or
deploy it into an existing one.

## Prerequisites

| Requirement | Check |
|---|---|
| Salesforce CLI (`sf`) 2.x | `sf --version` |
| Target org authenticated + aliased | `sf org display --target-org <alias>` |
| QuantumBit org with a **Price Revision** element configured in the Pricing Procedure | — |
| **Index Rates** table populated with records for lookup | — |
| **Price Revision Policies** configured | — |

Without the Price Revision element in the pricing procedure and the Index Rates /
Price Revision Policy data, the applied policy has nothing to resolve against.

## Deploy

The flow file sits at the repo root with no project. Wrap it, then deploy:

```bash
git clone https://github.com/jason-runnels-git/set-renewal-cpi-all-lines.git
cd set-renewal-cpi-all-lines

# 1. Create a minimal SFDX project layout
mkdir -p force-app/main/default/flows
cp Set_Renewal_Price_Uplift_Policy.flow-meta.xml force-app/main/default/flows/
cat > sfdx-project.json <<'JSON'
{
  "packageDirectories": [{ "path": "force-app", "default": true }],
  "sourceApiVersion": "62.0"
}
JSON

# 2. Validate, then deploy
sf project deploy start --source-dir force-app --target-org <alias> --dry-run
sf project deploy start --source-dir force-app --target-org <alias>
```

> Alternatively, drop the `.flow-meta.xml` into the `flows/` folder of an SFDX
> project you already have and deploy from there.

## Post-deploy steps

1. **Flow status:** deploys `Active` — confirm in Setup → Flows.
2. **Surface the flow.** It's a **Screen Flow**; add it as an **Action/Button** on
   the Quote (or wherever reps apply the renewal uplift) so it can be launched
   against a quote's lines.

## Verify

```bash
sf project deploy report --target-org <alias>
```
Open a renewal Quote with multiple lines → launch the flow → confirm the Price
Revision Policy is applied to **all** line items and reprices as expected.

## Known gotchas & limitations

- **No SFDX project in the repo** — wrap the loose flow file before deploying.
- **Data/config prerequisites are the real work:** the Price Revision pricing
  element, Index Rates records, and Price Revision Policies must exist first, or
  the uplift resolves to nothing.

## Safety

- Metadata-only deploy — no data is loaded or deleted by the deploy itself.
- The flow **modifies quote line pricing** when run; use a test quote first.
