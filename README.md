# fact-relationship

Which domains does the OMOP CDM `FACT_RELATIONSHIP` table link at your site? `CodeToRun.R` counts
`fact_relationship` rows per domain pair and writes the share of each pair (percentages only, no counts)
to `fact_relationships-aggregated-and-relative-data.csv`.

Background: [fact_relationship revisited in 2026](https://forums.ohdsi.org/t/fact-relationship-relationship-concept-id-table-fact-relationship-revisited-in-2026/26074) (OHDSI forums).

## Run it at your site

1. Edit the settings block at the top of `CodeToRun.R` (DBMS, CDM schema, connection). Leave everything
   below `# --- script starts here ----` as it is.
2. Run the script in R (needs `DatabaseConnector`, and `keyring` if you keep the credentials there).
3. Check the CSV before sharing it.

## As a multi-site study with OHDSI Study Studio (optional)

[OHDSI Study Studio](https://github.com/OHDSI/StudyStudio) can collect the CSVs from several sites with a
record of what each site ran, without any change to `CodeToRun.R`. [`study-studio/`](study-studio/) holds a
ready-made protocol, synthetic site data and sample StudyPacks, and its README explains how to use them.
