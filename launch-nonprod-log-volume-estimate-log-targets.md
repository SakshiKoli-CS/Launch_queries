QUESTION

Customer **Philips** (SF case **00059281**) enabled monitoring on their **Production** environment and, per their Monitoring Team, the **daily Launch log volume for Production is ~350 MB/day**. They now want to extend monitoring to their **Non-Production** environments but have no visibility into the log volume there, and asked whether Contentstack can **determine or estimate the daily log volume for the Non-Production environments**.

- **Log type:** Launch logs
- **Metric requested:** average daily log volume
- **Production volume:** ~350 MB/day
- **Non-Production scope:** all environments other than Production
- **Traffic segregation method:** Hostname
- **Stack API key:** `blte23630023bb36e19`

_Keywords: Launch log volume, daily log size estimate, non-production environments, log ingestion volume, Log Target, OpenTelemetry, observability platform, log retention, monitoring cost estimate, predict log volume._

ANSWER

## Can Launch provide / estimate Non-Production daily log volume? → No (not accurately)

Launch **cannot accurately provide or estimate** the daily log volume for Non-Production environments from its side, for two reasons:

1. **Format difference:** Launch stores logs **internally in a different format** than the format used when logs are **forwarded via Log Targets / OpenTelemetry**. So the volume observed in the customer's monitoring platform may **differ** from the volume stored within Launch — making it hard to predict the resulting ingestion volume.
2. **Limited retention:** Launch logs are **retained only for a limited period**, which does **not provide enough historical data** to compute an average daily log volume.

## Recommended way to measure it

- **Configure a Log Target** for the Non-Production environments and **monitor the ingestion volume directly in the observability platform.**
- This yields the **most accurate measurement**, based on the actual traffic and logging patterns of those environments (rather than an estimate).
- Support can assist with **Log Target configuration** if needed.

## Generalized guidance (for future similar queries)

- **"Can you tell me how much log volume environment X will generate?"** → Launch **cannot estimate this reliably**. The internally-stored format differs from the forwarded (Log Target/OTel) format, and log retention is short — there's no historical basis for an average.
- **The accurate path** is always: set up a **Log Target** on the target environment and read the **ingestion volume in the customer's own observability tool.**
- A known Production figure (e.g. ~350 MB/day) **does not reliably extrapolate** to other environments — traffic and logging patterns differ; measure each via its own Log Target.

## Status

- Guidance shared with the customer; acknowledged. Offer to help with Log Target setup extended.
