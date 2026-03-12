


| Name                                                                | Data Type | Value             | Allowed Values                           | Description                                     | Default Value |
| :------------------------------------------------------------------ | :-------- | :---------------- | :--------------------------------------- | :---------------------------------------------- | :------------ |
| <span target="chrome-datetime-format">Chrome-DateTime-Format</span> | text      | MM/DD/YY hh;mm;ss | [MM/DD/YY hh;mm;ss,YYYY, MM,DD hh;mm;ss] | Display format for datetime in global UI chrome |               |
| PlantID                                                             | integer   | 1                 | valid integers                           | ID value for the plant                          |               |
| Shift-Values                                                        | array     | array of integers | [1, 2, 3]                                | Available shift values                          |               |


Configuration Master - Options?


| Name                   | Data Type | Default Value | Allowed Values | Description                       |     |
| :--------------------- | :-------- | :------------ | :------------- | :-------------------------------- | :-- |
| Global-Lot-Numbers     | boolean   | false         | [True, False]  | 1 Lot for all Products            |     |
| Login-Clear-Program    | boolean   | false         | [True, False]  | for bad messages from indicator   |     |
| Login-Ind-Errors       | boolean   | false         | [True, False]  | For bad message from indicator    |     |
| Login-Force-Shiftentry | boolean   | false         | [True, False]  | Force entry of shift at login     |     |
| Login-Port             | integer   | 17800         | 0-8675309      | TCP port used for login server    |     |
| Log-Label-Error        | boolean   | false         | [True, False]  | Log errors in parsing label files |     |
| Login-Print-Process    | boolean   | false         | [True, False]  | Debug print Pgm process           |     |
