
- [ ] incomplete - need clearer definition of a device
- Printers
- Scales
# Business Definition

# Technical Definition

# Business Rules

# Properties

| name               | type      |     |     | Description                                                                 |
| ------------------ | --------- | --: | --- | --------------------------------------------------------------------------- |
| deviceId           | character |     |     |                                                                             |
| com-port           | integer   |     |     |                                                                             |
| networkId          | character |     |     |                                                                             |
| ip-address         | character |     |     |                                                                             |
| parity             | character |     |     |                                                                             |
| baud               | integer   |     |     |                                                                             |
| data-bits          | integer   |     |     |                                                                             |
| stop-bit           | integer   |     |     |                                                                             |
| timeout-ms         | integer   |     |     |                                                                             |
| model              | character |     |     |                                                                             |
| flow-control       | boolean   |     |     |                                                                             |
| input-buffer-size  | integer   |     |     |                                                                             |
| output-buffer-size | integer   |     |     |                                                                             |
| dtr-enable         | boolean   |     |     | Data Terminal Ready                                                         |
| description        | character |     |     |                                                                             |
| generic-printer    | logical   |     |     | Generic is like an HP line prtr, vs something like an I-class label printer |
| attached           | boolean   |     |     |                                                                             |