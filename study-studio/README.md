# Running fact-relationship with OHDSI Study Studio

[OHDSI Study Studio](https://github.com/OHDSI/StudyStudio) collects the result of `CodeToRun.R` from several
sites, together with a record of what each site ran. It is a single HTML page that works offline and sends
nothing over the network: use the [hosted build](https://ohdsi.github.io/StudyStudio/), or build
`dist/index.html` from its repository and open it from disk. Nothing has to be installed at the sites
beyond what `CodeToRun.R` already needs.

`CodeToRun.R` is used as it is. Study Studio pins the code **below** `# --- script starts here ----`:
every site must run that part unchanged, and may edit only the settings above it (DBMS, CDM schema,
connection).

## What is in this folder

| Path | What it is | Used by |
|---|---|---|
| `protocol/FACT_RELATIONSHIP_2026_protocol.zip` | The protocol: `study_config.json` and the pinned `CodeToRun.R` | Sites (*Site Mode*) |
| `protocol/study_config.json` | The same protocol on its own, as the reference for verification | Coordinator (*Multi-Site Aggregator*) |
| `sample-site-data/Site_A/`, `Site_B/`, `Site_C/` | `fact_relationships-aggregated-and-relative-data.csv` as three sites would write it. **Synthetic, not real data** | Trying *Site Mode* |
| `packs/Site_*_FACT_RELATIONSHIP_2026.studypack.zip` | The three outputs sealed by each site with its own copy of `CodeToRun.R` (PostgreSQL, SQL Server and Snowflake settings) | Trying *Multi-Site Aggregator* |

## Try it (about 3 minutes, no database needed)

1. **Seal a site's result.** Open Study Studio, stay in **Site Mode**, and type `Site_A` as the Site ID.
   - Step 1: drop `protocol/FACT_RELATIONSHIP_2026_protocol.zip`.
   - Step 2: drop `sample-site-data/Site_A/fact_relationships-aggregated-and-relative-data.csv`.
     The CSV is audited for small cells and direct identifiers. Because it holds percentages only, a
     warning says that small cells cannot be checked here.
   - Step 3: **Download StudyPack**. You get `Site_A_FACT_RELATIONSHIP_2026.studypack.zip`, with the SHA-256
     of every file in its `manifest.json`.
2. **Verify the sites.** Reload the page (each role starts from a fresh page), switch to
   **Admin Mode → Multi-Site Aggregator**, drop `protocol/study_config.json` as the reference, then drop the
   three files in `packs/` (or your own pack from step 1 in place of Site_A's).
   All three are recommended for adoption: their settings blocks differ, the code below the marker is the same.
3. **Get the combined result.** In **Export Hub → Merged results**, download one CSV with the three sites'
   rows under a `site_id` column. The **Audit report** (Markdown or PDF) records what was received,
   what was checked and what was decided.

To see a site flagged, open a pack, change a value in its CSV or a line of `CodeToRun.R` below the marker,
re-zip it and drop it in the Aggregator: it is shown as tampered, or as a protected-section mismatch.

## Running a real study

**Coordinator**
1. In **Admin Mode → Protocol Issuer**, drop this repository's ZIP (GitHub "Download ZIP") or `CodeToRun.R`.
   It is recognised as an *R script study*, and `# --- script starts here ----` is suggested as the line
   to pin below. Fill in the Study ID and **Issue protocol**.
2. Send `<studyId>_protocol.zip` to the sites. Keep `study_config.json` for verification.

**Each site**
1. Edit the settings block of `CodeToRun.R` and run it as usual.
2. In **Site Mode**: enter your Site ID, drop the protocol ZIP, drop the CSV, check the audit, and
   **Download StudyPack**.
3. Send the `.studypack.zip` to the coordinator by whatever channel you normally use.

**Coordinator**
1. In **Multi-Site Aggregator**, drop `study_config.json` and the packs. Each pack is checked for files
   changed after sealing and for code below the marker that differs from the protocol. Adopt or exclude
   each site; every decision goes into a hash-chained audit trail.
2. In **Export Hub**, download the merged CSV and the audit report.

## Good to know

- **Percentages only.** The output has no counts, so Study Studio cannot check it for small cells; a rare
  domain pair shows up as a small non-zero percentage. If that matters for your sites, suppress small
  counts in `CodeToRun.R` before the percentages are computed.
- **Change the script, reissue the protocol.** If the code below the marker changes, issue a new protocol.
  Packs sealed with the old one will not match it.
- **What a pack proves.** A pack shows which script and which CSV the site sealed, and that neither changed
  afterwards. It does not prove that the CSV came from running that script: Study Studio has no run record
  for plain R scripts.
- These files were produced with OHDSI Study Studio at commit
  [`53e1d47`](https://github.com/OHDSI/StudyStudio/commit/53e1d47).
