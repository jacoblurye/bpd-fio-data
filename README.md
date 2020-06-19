# Boston Police Department Field Interrogation and Observation Dataset (2016-2019)

This repository contains data from BPD's FIO [dataset](https://data.boston.gov/dataset/boston-police-department-fio), merged across years and cleaned in two analysis-ready CSV files.

Each row in the `fio_contacts.csv` file pertains to a single police "FIO", which could entail anything from observing people from afar to stopping, frisking, and arresting them. Each row in the `fio_people.csv` file pertains to a person involved in one of those FIOs. Each stop in `fio_contacts.csv` has a unique identifier, `fc_num`, that links to the people in `fio_people.csv` who were contacted.

The code used to generate this file is presented in the jupyter notebook included in this repo, and the specific transformations that that code performs are documented below.

_Disclaimer_: I've done my best to preserve or clarify the original meaning of all data points through cleaning and value reconciliation, but please assess for yourself whether you consider my approach valid before drawing conclusions from any analysis performed with this dataset. And if you _do_ find something fishy, don't hesistate to file an issue.

## Data-cleaning operations

- Combine the `frisked` and `searchperson` columns from the _contacts_ table into one column called `fc_involved_frisk_or_search`,
  and disambiguate it from a related column in the _people_ table by renaming the `frisk/search` column to `person_frisked_or_searched`.
  As noted on data.boston.gov, the `frisked` and `searchperson` columns (from the "New RMS" data system) indicate whether any of the
  individuals stopped in a given field contact was frisked, whereas `frisk/search` (from the "Mark43" data system) indicates whether
  a particular person involved in a field contact was frisked.
- Clean up and reconcile vehicle-related field values from the _contacts_ table. Across the different data systems, different text values
  were used to represent identical or overlapping concepts (e.g. "LT. BLUE" and "light blue", "Suv (sport Utility Vehicle)"
  and "SUV or Utility Van"). Also, drop the `vehicle_style` column, which is basically a noisier version of the `vehicle_type` column.
- Convert the `contact_date` column in the _contacts_ table to datetime,
  and drop a duplicate column from the _people_ table.
- Remove extra whitespace from the `contact_officer_name` column in the _contacts_ table
  to reduce accidental duplication. Also, put officer's first names first.
- Merge the _contacts_ table's `contact_reason` column into the `narrative` column. `contact_reason` serves the same purpose as `narrative`,
  just for the older system. Uppercase both columns for consistency.
- There are lots of typos and inconsistencies in the _contacts_ table's `city` column, so fix them.
  Warning: current solution is not robust to _new_ typos, should the data change.
- Remove implausibly high values in the _people_ table's `age` column. Also, convert string values to float.
- Merge the _people_ table's older `complexion` column into the newer `skin_tone` column.
- Drop the `deceased` column from the _people_ table, since it doesn't exist in the older system, and since no one
  is marked deceased in this dataset.
- Reconcile the _people_ table's `hair_style` values that seem to mean the same thing.
- Reconcile _people_ table `race` values that seem to mean the same thing.
- Fix recurrent typo in the _contacts_ table `state` column.
- Bucket _contacts_ table `stop_duration` column values listed as minutes and remove likely typos.
- Combine the _contact_ table's `street` and `streetaddr` columns into single `street` column, and trim extra digits from `zip`.
- The values `''`, `'NULL'`, and `'None'` are used to signify no value was entered in various fields across the dataset.
  Replace these values with `np.nan`.
