# Resource Notes

## Data Generation

### FOCUS

- [General Spec info](https://focus.finops.org/focus-specification/)
- [Column Library](https://focus.finops.org/focus-columns/)
- [Use cases, queries](https://focus.finops.org/use-cases)
- [Sample Data (github) - Granted this is still 1.0, and csv](https://github.com/FinOps-Open-Cost-and-Usage-Spec/FOCUS-Sample-Data)

### AWS

#### [FinOps Foundation Guide for FOCUS Datasets](https://focus.finops.org/get-started/aws/)

- Export Data
  - [FOCUS Data export](https://docs.aws.amazon.com/cur/latest/userguide/dataexports-create.html) (preferred)
    - Also specifying selections
      - e.g. parquet format
      - Daily vs. Hourly
      - bucket prefix
      - FOCUS vs. 1.0 vs 1.2
  - CloudFormation template (not preferred)

- Creating a [standard export](https://docs.aws.amazon.com/cur/latest/userguide/dataexports-create-standard.html)
- [Conformance Gaps for FOCUS 1.2](https://docs.aws.amazon.com/cur/latest/userguide/table-dictionary-focus-1-2-aws-conformance.html)
  - Guidance on export columns with missing or misleading data.
