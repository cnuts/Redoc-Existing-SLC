
- [ ] incomplete - item

# Business Definition

# Technical Definition

# Business Rules

# Properties


| name                          | type      | length | Description                                                                                                                                                     |
| ----------------------------- | --------- | ------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| extra-tare-type               | character |        | Additional tare type: N=None,W=Weight,P=Percent                                                                                                                 |
| item-code                     | integer   |        | Item Code in the range 00001-99999.                                                                                                                             |
| max-weight                    | decimal   |        | Maximum weight to allow printing a label.                                                                                                                       |
| min-label-weight              | decimal   |        | Used with PackType of 'V', for Variance; initially for Conagra; Used to calc Label Wgt: if wgt < MinLblWgt,                                                     |
| min-weight                    | decimal   |        | Minimum weight to allow printing a label.                                                                                                                       |
| pack-type                     | character |        | Package Weight Type: C=Catch, U=Unipak, F=Fixed, K=Key-in, V=Variance                                                                                           |
| sell-by-format                | character |        | Sell by date print and display format                                                                                                                           |
| weight-round-or-truncate      | character |        | Should we round or truncate weights                                                                                                                             |
| weight-units                  | character |        | Weight Unit type: LB or KG                                                                                                                                      |
| description1                  | character |        |                                                                                                                                                                 |
| description2                  | character |        |                                                                                                                                                                 |
| short-description1            | character |        |                                                                                                                                                                 |
| short-description2            | character |        |                                                                                                                                                                 |
| short-description3            | character |        |                                                                                                                                                                 |
| standard-weight               | decimal   |        | Standard weight to print for fixed and unipak Item.                                                                                                             |
| weight-round-digit            | integer   |        |                                                                                                                                                                 |
| label-file                    | character |        | Label file name                                                                                                                                                 |
| item-date-format              | character |        |                                                                                                                                                                 |
| sell-by-offset                | integer   |        | Number of days from the kill date to offset the sell by date                                                                                                    |
| tare-weight                   | decimal   |        | Tare value of packaging                                                                                                                                         |
| weight-tare1                  | decimal   |        | Weight tare value to be added to box tare value                                                                                                                 |
| weight-tare2                  | decimal   |        | Weight tare value to be added to box tare value                                                                                                                 |
| weight-tare3                  | decimal   |        | Weight tare value to be added to box tare value                                                                                                                 |
| weight-tare4                  | decimal   |        | Weight tare value to be added to box tare value                                                                                                                 |
| weight-tare5                  | decimal   |        | Weight tare value to be added to box tare value                                                                                                                 |
| weight-tare6                  | decimal   |        | Weight tare value to be added to box tare value                                                                                                                 |
| weight-tare7                  | decimal   |        | Weight tare value to be added to box tare value                                                                                                                 |
| weight-tare8                  | decimal   |        | Weight tare value to be added to box tare value                                                                                                                 |
| precent-tare-1                | decimal   |        | Percentage tare value to be added to box tare value                                                                                                             |
| precent-tare-2                | decimal   |        | Percentage tare value to be added to box tare value                                                                                                             |
| precent-tare-3                | decimal   |        | Percentage tare value to be added to box tare value                                                                                                             |
| precent-tare-4                | decimal   |        | Percentage tare value to be added to box tare value                                                                                                             |
| precent-tare-5                | decimal   |        | Percentage tare value to be added to box tare value                                                                                                             |
| precent-tare-6                | decimal   |        | Percentage tare value to be added to box tare value                                                                                                             |
| precent-tare-7                | decimal   |        | Percentage tare value to be added to box tare value                                                                                                             |
| precent-tare-8                | decimal   |        | Percentage tare value to be added to box tare value                                                                                                             |
| text1                         | character |        | Text field 1                                                                                                                                                    |
| text2                         | character |        | Text field 2                                                                                                                                                    |
| text3                         | character |        | Text field 3                                                                                                                                                    |
| text4                         | character |        | Text field 4                                                                                                                                                    |
| text5                         | character |        | Text field 5                                                                                                                                                    |
| text6                         | character |        | Text field 6                                                                                                                                                    |
| text7                         | character |        | Text field 7                                                                                                                                                    |
| text8                         | character |        | Text field 8                                                                                                                                                    |
| text9                         | character |        | Text field 9                                                                                                                                                    |
| text10                        | character |        | Text field 10                                                                                                                                                   |
| max-label-weight              | decimal   |        | Used with PackType of 'V', for Variance; initially for Conagra; Used to calc Label Wgt: if wgt > MaxLblWgt, label wgt = MaxLblWgt. (still can't exceed Max Wgt) |
| post-tare                     | decimal   |        | Usually Ice or Packing material                                                                                                                                 |
| exact-items-per-case-required | bool      |        |                                                                                                                                                                 |
| date-ai                       | character |        | UCC Std for barcodes - date AI                                                                                                                                  |
| date-ai-date                  | character |        | Date to be used in conjunction with DateAI                                                                                                                      |
| offset2                       | integer   |        | 2nd offset to be used with AI dates for barcoding                                                                                                               |
| prod-date-format              | character |        |                                                                                                                                                                 |
| var-format                    | character |        |                                                                                                                                                                 |
| var-offset                    | integer   |        |                                                                                                                                                                 |
