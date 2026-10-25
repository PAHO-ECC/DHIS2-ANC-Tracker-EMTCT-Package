# DHIS2 ANC Tracker — EMTCT Package

An importable DHIS2 metadata package for an **Antenatal Care (ANC) tracker program** built around
**EMTCT** — the Elimination of Mother-To-Child Transmission of HIV, syphilis and hepatitis B.
It captures booking information, antenatal screening (HIV, syphilis, hepatitis B and other
investigations), partner testing and treatment, syphilis treatment follow-up, referral and the
pregnancy outcome, and ships with the program rules and program indicators needed to monitor
EMTCT performance.

Exported from DHIS2 **2.42.4.1**. Should import into any 2.41+ instance.

## Contents

| File | What it is |
|---|---|
| `metadata.json` | The complete DHIS2 metadata bundle (see breakdown below). Import it as-is through **Import/Export → Metadata import**, or `POST /api/metadata`. |
| `LICENSE` | GNU General Public License v3.0. |
| `README.md` | This file. |

### What's inside `metadata.json`

| Object type | Count | Notes |
|---|---|---|
| Program | 1 | `ANC` — tracker program (`WITH_REGISTRATION`). `onlyEnrollOnce: false`, so one woman can be enrolled once per pregnancy. |
| Program stage | 1 | `ANC` — a single, non-repeatable stage that holds the whole pregnancy episode (booking, follow-up visits 2–4, results, treatment, outcome). Auto-generated on enrollment. |
| Program stage sections | 7 | Booking Information · Booking Investigations · Second/Subsequent Visit · Partner Testing & Treatment · Treatment · Referral · Outcome |
| Data elements | 71 | Clinical fields for the stage, including gravida/para/abortions, LMP/EDD/gestational age, haemoglobin, blood group, Hb electrophoresis, HIV/syphilis/hepatitis B tests (1st, 2nd and repeat tests with dates, types, results, titres), chlamydia, gonorrhoea, rubella, vaccinations, partner testing and treatment, syphilis treatment dates/details, referral, and pregnancy outcome. |
| Tracked entity type | 1 | `Person / Patient` |
| Tracked entity attributes | 12 | Name, date of birth, National ID, HMIS ID, ANC Register Number, Referral Hospital Registration Number, address, parish, nationality, phone, email, religion. |
| Option sets / options | 14 / 260 | Test results, blood group, Hb electrophoresis genotypes, rubella results, syphilis screening/confirmatory test types, titres, treatment drugs, gestational-age source, pregnancy outcome, nationality (195 countries, A–Z), parishes, religion. |
| Program rules / actions / variables | 46 / 84 / 37 | Gestational age at booking and current gestational age (from LMP, with LMP back-calculated from EDD when only the EDD is known), hide-and-clear logic for test fields that don't apply (e.g. repeat syphilis tests only when the first result is reactive), status displays ("result pending / missing"), and overdue-visit warnings. |
| Program indicators | 37 | EMTCT and ANC monitoring: tests done, results received and positivity for HIV, syphilis and hepatitis B; syphilis cases treated with penicillin G and retested after treatment; booking before/after 20 weeks; age-at-booking bands; gravidity/parity groups; anaemia at booking; women with 4 ANC visits; mean age at booking; overdue visits. |
| Attributes | 5 | Metadata attributes for terminology codes: ICD-10, ICD-11, SNOMED CT, LOINC and FHIR R4. Many data elements carry these codes, with links to public LOINC / NHS SNOMED term browsers. |
| Category model | 1 each | The DHIS2 `default` category, option, combo and option combo. |

## Importing

1. **Dry-check first** with `importMode=VALIDATE` (not `dryRun=true` — on some DHIS2 2.42 builds
   `dryRun` is ignored and the import is committed):
   ```bash
   curl -u USER:PASS -H "Content-Type: application/json" \
     -X POST "https://YOUR-DHIS2/api/metadata?importMode=VALIDATE&importStrategy=CREATE_AND_UPDATE&atomicMode=NONE" \
     --data-binary @metadata.json
   ```
2. If the validation report is clean, run the same call with `importMode=COMMIT`.
3. Assign **sharing** for the program, stage, data elements, attributes and option sets to the
   right user groups (sharing is not included in this package). A program the users can't see,
   or data elements without data-read sharing, will render as a broken Capture form.
4. Assign the program to your **organisation units** — the package ships with none.
5. **Replace the parish list** with your own administrative subdivisions (see below).
6. Run analytics so the program indicators have data.

## Adapting to your setting

- **`GEN - Parishes`** ships with 11 sample parish names. Edit the option set to hold your own
  administrative subdivisions (parishes, districts, regions…) before going live.
- **`ANC - Referral Hospital Registration Number`** is a free-text field for the patient's number
  at the national/referral hospital where deliveries or specialist care happen. Rename it to your
  hospital's name if that reads better for your users.
- **`GEN - HMIS ID`** holds the patient's identifier in your health management information
  system, if you have one.

## What was removed or changed before publishing

- **All user information was stripped.** Every `createdBy` and `lastUpdatedBy` block (user id,
  name, username), every `sharing` block (owner, user and user-group access), and every
  `favorites` / `favorite` flag. Objects will be owned by whoever imports them, and sharing
  defaults to the target instance's settings.
- **Organisation unit assignments were removed** from the program, so the package carries no
  facility hierarchy.
- **The source instance's `system` block** (system id, build revision, export date) was removed;
  it isn't needed for import.
- **Country-specific naming was generalized**: object names, codes and descriptions no longer
  carry a country prefix, and the hospital and patient-ID attributes use generic names. UIDs are
  unchanged.

## Notes

- UIDs are preserved, so re-importing over an instance that already has this program updates it
  in place rather than creating duplicates.
- The `ANC - Pregnancy Outcome` data element is **compulsory**: an ANC event can't be marked
  complete until an outcome (Live birth / Miscarriage / Stillbirth / Transferred / Lost to
  follow-up) is chosen. This is how the program closes out a pregnancy, so that a later
  pregnancy can be enrolled without hitting DHIS2's "already has an active enrollment" block.

## License

GNU General Public License v3.0 — see [`LICENSE`](LICENSE).
