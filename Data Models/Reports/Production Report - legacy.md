
## 1. Overview

### What Is the Product List Report?

The **Product List Report** is a comprehensive product master data listing that displays all products configured in the manufacturing system with their weight specifications and operational parameters. It answers the critical question: _"What products are available in the system, what are their weight specifications, and what are their operational characteristics?"_

The report queries the **Product** table directly and presents all product master records in a formatted table, allowing production supervisors, product managers, quality personnel, and system administrators to view complete product configuration information.

### Why It Exists (Business Value)

1. **Product Configuration Verification**: Verify that all products are properly configured with correct weight specifications
2. **Specification Documentation**: Create printed or digital documentation of all product weight specifications for compliance and training
3. **Production Reference**: Allow operators to quickly look up product specifications during production
4. **System Audit Trail**: Maintain documented record of all products in the system for regulatory compliance
5. **Labeling Information**: Display label file assignments for each product
6. **Cross-functional Communication**: Provide product specifications to quality, production, and packaging teams
7. **Change Control**: Track product configuration changes by running reports at different times

### Report Characteristics

| Attribute               | Value                                                                        |
| ----------------------- | ---------------------------------------------------------------------------- |
| **UI Entry Point**      | Main Menu → Reports → Product List (s-repmenu.w button: btnProductList)      |
| **Implementation File** | sobjects/s-repviewer.w (CreateProductReport procedure)                       |
| **Report Type**         | Master data listing (no parameters)                                          |
| **Data Source**         | Product table (all records)                                                  |
| **Sort Order**          | By ProductCode (ascending)                                                   |
| **Output Format**       | Formatted text report, paginated for display and printing                    |
| **Display Format**      | Editor widget in s-repviewer.w UI viewer                                     |
| **Print Format**        | Text file (reports\pproduct.txt) - page breaks every 66 lines for printer    |
| **Typical Users**       | Product managers, production supervisors, QA analysts, system administrators |

---

## 2. Data Source: Product Table

### Core Table Definition

The Product List Report queries the **Product** table, which contains the master data for all products manufactured by the facility. Each Product record represents a unique product code with its associated specifications.

**Key Fields Used in Report:**

| Field           | Type      | Length | Purpose                                            |
| --------------- | --------- | ------ | -------------------------------------------------- |
| **ProductCode** | Character | 5      | Unique product identifier (primary key, sort key)  |
| **ShortDesc1**  | Character | 8      | First description line (part 1 of 3)               |
| **ShortDesc2**  | Character | 8      | Second description line (part 2 of 3)              |
| **ShortDesc3**  | Character | 8      | Third description line (part 3 of 3)               |
| **PackType**    | Character | 1      | Packing type code (e.g., 'C' = case, 'P' = pallet) |
| **MinWgt**      | Decimal   | 7,2    | Minimum acceptable weight specification            |
| **StdWgt**      | Decimal   | 7,2    | Standard/target weight specification               |
| **MaxWgt**      | Decimal   | 7,2    | Maximum acceptable weight specification            |
| **WgtUnits**    | Character | 1      | Weight unit of measure ('L' = lbs, 'K' = kg)       |
| **LabelFile**   | Character | 8      | Associated label file name or code                 |

**Query Logic:**

```progress
FOR EACH Product NO-LOCK
    BREAK BY ProductCode
```

The report performs a simple forward scan of ALL Product records (no filtering), breaking at each ProductCode for proper page separation.

---

## 3. Report Output Format

### 3.1 Display Format (Editor Widget)

The report generates formatted output displayed in the s-repviewer.w editor widget (ediReport: 127 chars wide × 17 lines visible).

**Header Structure:**

```
                                     Pk    Min      Std      Max     Label
 Prod  Description            Ty    Wgt      Wgt      Wgt UOM File
-----  --------  --------  --------  -  --------  --------  --------  -  --------
```

**Data Line Example:**

```
12301  Apple    Sauce     Organic    C   7.50      8.00      8.50     L  LABEL01
```

### 3.2 Column Layout (Positional Formatting)

The report uses fixed-position formatting (column positions specified with AT clause):

| Column Start | Width | Field       | Format Code        |
| ------------ | ----- | ----------- | ------------------ |
| 1            | 5     | ProductCode | '99999'            |
| 7            | 8     | ShortDesc1  | 'X(8)'             |
| 15           | 8     | ShortDesc2  | 'X(8)'             |
| 23           | 8     | ShortDesc3  | 'X(8)'             |
| 32           | 1     | PackType    | 'X(1)' (uppercase) |
| 34           | 8     | MinWgt      | 'zZZZ9.99'         |
| 43           | 8     | StdWgt      | 'zZZZ9.99'         |
| 52           | 8     | MaxWgt      | 'zZZZ9.99'         |
| 61           | 1     | WgtUnits    | 'X(1)' (uppercase) |
| 63           | 8     | LabelFile   | 'X(8)'             |

**Format Specifications:**

- `'99999'`: Right-justified integer (spaces for leading zeros)
- `'X(n)'`: Left-justified character string
- `'zZZZ9.99'`: Decimal with zero suppression (spaces for leading zeros), decimal precision 2
- `CAPS()`: Function called on PackType and WgtUnits to ensure uppercase display

### 3.3 Pagination (Interactive Display)

The report includes automatic pagination:

- **Page Size**: 17 display lines (defined as {&EDI_PAGE_SIZE})
- **Line Counter**: Resets at page boundaries for proper formatting
- **Page Navigation**: Buttons available if report exceeds 20 lines (17 + 3):
  - Top button: Jump to first page
  - Up button: Previous page
  - Down button: Next page
  - Bottom button: Jump to last page

**Page Padding:**

- After report generation, blank lines are added to complete the final page
- This ensures proper pagination when printing or scrolling

---

## 4. Report Generation Logic (CreateProductReport Procedure)

### 4.1 Initialization & Variables

```progress
/* Initialize line counter for pagination */
intLineCounter = 1

/* Open output stream to file */
OUTPUT STREAM s-widstream TO "reports\product.txt"

/* Initialize header lines with localized labels */
v-line-1 = "Pk    Min      Std      Max     Label" (formatted with translations)
v-line-2 = "Prod  Description            Ty    Wgt      Wgt      Wgt UOM File" (formatted)
```

**Localization Keys Used:**

- Reports.ProdList.Pack (abbreviation for "Pack Type")
- Reports.ProdList.Min (abbreviation for "Min Weight")
- Reports.ProdList.Std (abbreviation for "Std Weight")
- Reports.ProdList.Max (abbreviation for "Max Weight")
- Reports.ProdList.UOM (abbreviation for "Unit of Measure")
- Reports.ProdList.Label (label for "Label File")
- Reports.ProdList.Prod (abbreviation for "Product")
- Reports.ProdList.Desc (abbreviation for "Description")
- Reports.ProdList.Type (abbreviation for "Type")
- Reports.ProdList.Wgt (repeats for Weight fields)
- Reports.ProdList.File (abbreviation for "File")

### 4.2 Report Iteration & Page Breaks

```progress
DO WITH FRAME {&FRAME-NAME}:
    intLineCounter = 1
    FOR EACH Product NO-LOCK
        BREAK BY ProductCode:

        /* Print headers on first iteration */
        IF (intLineCounter = 1) THEN DO:
            PUT STREAM "blank line" SKIP
            PUT STREAM v-line-1 AT 32 SKIP  /* Weight headers start at col 32 */
            PUT STREAM v-line-2 SKIP         /* Product description headers */
            PUT STREAM underline row SKIP    /* Separator */
        END

        /* Output product data line */
        PUT STREAM unformatted
            ProductCode AT 01
            ShortDesc1  AT 07
            ShortDesc2  AT 15
            ShortDesc3  AT 23
            PackType    AT 32  (uppercase)
            MinWgt      AT 34  (decimal formatted)
            StdWgt      AT 43  (decimal formatted)
            MaxWgt      AT 52  (decimal formatted)
            WgtUnits    AT 61  (uppercase)
            LabelFile   AT 63
            SKIP

        /* Increment line counter and reset at page boundary */
        intLineCounter = intLineCounter + 1
        IF (intLineCounter > EDI_PAGE_SIZE) THEN
            intLineCounter = 1
    END  /* FOR EACH Product */
END  /* DO WITH FRAME */
```

### 4.3 File Output & Display

After report generation completes:

```progress
/* Close output stream */
OUTPUT STREAM s-widstream CLOSE

/* Insert file contents into editor widget for display */
ediReport:INSERT-FILE("reports\product.txt")

/* Calculate padding lines needed to complete final page */
intLineCounter = ediReport:NUM-LINES MODULO {&EDI_PAGE_SIZE}

/* Clear editor and re-append with padding */
ediReport:SCREEN-VALUE = ''
OUTPUT STREAM s-widstream TO "reports\product.txt" APPEND

/* Add blank lines to complete page */
DO intLineCounter = intLineCounter + 1 TO {&EDI_PAGE_SIZE}:
    PUT STREAM s-widstream " " SKIP
END

OUTPUT STREAM s-widstream CLOSE
ediReport:INSERT-FILE("reports\product.txt")
```

---

## 5. Relationships to Other Concepts

### Product Table (Primary Data Source)

- **Relationship**: Report queries Product table directly
- **Link Fields**: ProductCode (primary key)
- **Cardinality**: One Product record → one line in report
- **Fields Displayed**: ProductCode, ShortDesc1-3, PackType, MinWgt, StdWgt, MaxWgt, WgtUnits, LabelFile

### ItemSerial Table (Production Records)

- **Relationship**: ItemSerial records reference Product via ItemCode
- **Link Fields**: Product.ProductCode = ItemSerial.ItemCode
- **Use Case**: Verify that products in production match product configuration
- **When Relevant**: Cross-referencing actual production against configured products

### Serial Table (Case/Batch Master)

- **Relationship**: Serial records contain production of multiple products
- **Link Fields**: Serial.ProductCode references Product
- **Use Case**: Ensure all products in production are configured in master list
- **When Relevant**: Auditing production against product catalog

### Modify Table (Production-time Adjustments)

- **Relationship**: Modify records may override Product specifications during production
- **Link Fields**: Modify.ProductCode references Product.ProductCode
- **Data Impact**: Actual production weights may differ from Product standard weights
- **When Relevant**: Understanding differences between spec and actual production

### Goals Table (Production Targets)

- **Relationship**: Goals tied to Products for production target tracking
- **Link Fields**: Goals.ProductCode references Product (conceptual)
- **Use Case**: Compare production targets with product specifications
- **When Relevant**: Production planning and goal achievement analysis

### ConfigOptionMaster Table (System Configuration)

- **Relationship**: Not directly related but may reference product-related defaults
- **Use Case**: Product list may be filtered by configuration options in future versions
- **When Relevant**: Multi-facility operations with different product catalogs

---

## 6. Business Rules

### Report Scope & Selection Rules

**Rule 6.1: Universal Product Listing**

- Report displays ALL products in the Product table
- No filtering or parameter selection
- Every configured product appears in the report
- More recent products appear in sorted order by ProductCode

**Rule 6.2: Product Code Primary Sort**

- Products sorted in ascending order by ProductCode
- Two products with identical ProductCode would cause duplicate entries (unlikely in production)
- Maintains logical sequence for reference and searching

**Rule 6.3: No Deactivation Filtering**

- All products appear regardless of whether they are currently in production use
- Retired or obsolete products remain visible
- Allows audit trail of all products ever configured

### Data Presentation Rules

**Rule 6.4: Description Concatenation**

- Three separate description fields (ShortDesc1, ShortDesc2, ShortDesc3) appear side-by-side
- Combined they provide up to 24 characters of product description
- Each field is independently formatted, allowing different content
- Example: "Apple Sauce Organic " → displays as three separate 8-char columns

**Rule 6.5: Weight Specification Formatting**

- MinWgt, StdWgt, MaxWgt formatted as decimal with 2 decimal places
- Format 'zZZZ9.99' suppresses leading zeros but shows decimal point
- Examples: "7.50", "0.75", "10.00"
- Negative weights display with minus sign (though invalid in practice)

**Rule 6.6: Pack Type Display**

- PackType field contains single character (e.g., 'C' for case, 'P' for pallet)
- Displayed in UPPERCASE to ensure consistency
- Should correspond to valid pack types in configuration

**Rule 6.7: Weight Units Display**

- WgtUnits field (single character) shown in UPPERCASE
- Typical values: 'L' (pounds/lbs), 'K' (kilograms/kg)
- Determines how weight values are interpreted in production

**Rule 6.8: Label File Identification**

- LabelFile field (up to 8 characters) identifies associated label template
- Matches filenames or file codes in labeling system
- May be blank if no specific label is assigned
- Used to trigger label printing for products

### Pagination & Output Rules

**Rule 6.9: Fixed Page Size**

- Display pages contain exactly 17 content lines
- Printer pages contain exactly 66 lines (standard printer page)
- Page breaks initiated at line boundaries for clean pagination

**Rule 6.10: Header Repetition**

- Column headers printed on first page only (intLineCounter = 1)
- Does NOT repeat on subsequent pages in current implementation
- Enhancement opportunity: Could add header rows on each new page

**Rule 6.11: Page Completion**

- Blank lines added to complete final page after report generation
- Ensures proper page breaks when printing
- Lines added to match EDI_PAGE_SIZE boundary

**Rule 6.12: No Data Filters or Sub-totals**

- Report shows detail lines only (no summary lines)
- No subtotals, totals, or grouping subtotals
- One line per product, no blank lines between groups

### File & Output Rules

**Rule 6.13: Output File Locations**

- Display report file: `reports\product.txt`
- Print report file: `reports\pproduct.txt`
- Both files contained in reports subdirectory relative to application root

**Rule 6.14: Editor Display**

- Report contents loaded into ediReport widget (editor control)
- Editor is READ-ONLY (user cannot modify displayed content)
- Editor displays 127 characters wide × 17 lines visible area

**Rule 6.15: Page Navigation Visibility**

- Page navigation buttons (Top, Up, Down, Bottom) hidden if report ≤ 20 lines
- Buttons enabled and visible if report > 20 lines
- Users can scroll through report using navigation buttons

---

## 7. Common Queries & Data Interpretation

### Query 1: Verify Product Configuration Completeness

**Question**: "Is product 12301 properly configured in the system?"

**Answer from Report**:

1. Locate the row with ProductCode = 12301
2. Verify all fields are populated:
   - Description fields (ShortDesc1, ShortDesc2, ShortDesc3) contain product name/type
   - PackType has a value (e.g., 'C')
   - Weight specifications (MinWgt, StdWgt, MaxWgt) are reasonable values
   - WgtUnits is populated ('L' or 'K')
   - LabelFile has an assigned template name

**Example Interpretation**:

```
12301  Apple    Sauce     Organic    C   7.50      8.00      8.50     L  LABEL01

✓ All fields populated
✓ Product is properly configured
✓ Ready for production use
```

---

### Query 2: Cross-check Production Weights Against Specifications

**Question**: "Are the weights we're producing for item 12302 within the configured specification?"

**Answer from Report**:

1. Find product 12302 in report
2. Note the MinWgt, StdWgt, MaxWgt values
3. Compare actual production weights from Item Production Report to these specs

**Example**:

```
Product List Report:
12302  Applesauce - - C   7.25      8.00      8.75     L  LABEL02

Item Production Report (for same day):
Item 12302: Average weight 8.15 oz

Interpretation:
Specification: 7.25 - 8.75 oz
Actual: 8.15 oz
STATUS: ✓ WITHIN SPEC (8.15 is between 7.25 and 8.75)
Margin: 0.15 oz above standard
```

---

### Query 3: Identify Label Assignment for Packaging

**Question**: "Which label template should we use for product 15401?"

**Answer from Report**:

1. Locate product 15401 in the list
2. Read the LabelFile column value
3. Use that filename to retrieve label template from labeling system

**Example**:

```
15401  Sauce    Cup      Small      C   3.00      3.50      4.00     L  LABEL05

ACTION: Use label template "LABEL05" for product 15401 packaging
```

---

### Query 4: Audit Product Catalog Completeness

**Question**: "Is our entire product catalog configured in the system?"

**Process**:

1. Run Product List Report
2. Count total number of products shown
3. Compare to expected product count from master list
4. Identify any missing products (should appear but don't)

**Expected Outcome Report**:

```
Total Products Found: 47
Expected Products: 50
Missing: 3 products (IDs: 18901, 19002, 19105)

ACTION: Contact product management to add missing products to system
```

---

### Query 5: Identify Products by Weight Unit

**Question**: "Which products are configured for kilogram (KG) weights vs pounds (LB)?"

**Process**:

1. Review entire Product List Report
2. Scan WgtUnits column for 'K' vs 'L' values
3. Group products by weight unit

**Example Categorization**:

```
POUND-based Products (WgtUnits = L):
  12301, 12302, 12305, 15401, 15402

KILOGRAM-based Products (WgtUnits = K):
  21001, 21002, 21005

ACTION: Ensure production scales are configured for correct weight units
        Verify labeling templates use correct units
```

---

## 8. Printing & Output Options

### Display Viewing (s-repviewer.w UI)

When the Product List Report is generated:

1. Report creates file: `reports\product.txt`
2. Contents loaded into ediReport editor widget
3. User sees formatted report in editor
4. Can scroll through pages using navigation buttons
5. Can print directly from viewer using "Print" button

### Print Output Format

The report includes a dedicated print generation called from the "Print" button:

**Print Procedure**: `ip-printreport`

**Key Differences from Display:**

- Print output file: `reports\pproduct.txt`
- Page size: 66 lines per page (standard printer format)
- Includes page breaks (using `PAGE` statement)
- Proper spacing for 8.5×11 inch paper
- Column positions adjusted for printed output

**Print Header Formatting**:

```
  Pk    Min      Std      Max     Label
  Prod  Description            Ty    Wgt      Wgt      Wgt UOM File
-----  ---------  ---------  --------  -  --------  --------  --------  -  --------
```

**Print Data Lines**:

- Same content as display but with adjusted column positions
- All data from Product table displayed on printed output
- Page breaks every 66 lines for proper printer pagination

---

## 9. Integration Points

### 1. Link to Item Production

**Question**: "Which products were produced yesterday according to the Item Production Report?"

**Integration Path**:

1. Run Product List Report → Get all configured products
2. Run Item Production Report → Get yesterday's production items
3. Cross-reference ItemCode in Item Production Report against ProductCode in Product List
4. Identify which products were produced

**Use Case**: Production summary by product type

---

### 2. Link to Production Goals

**Question**: "What is the daily production goal for each product?"

**Integration Path**:

1. Display Product List Report
2. For each product, look up Goals table entries
3. Match Product.ProductCode = Goals.ProductCode
4. Display goal quantities alongside product list

**Use Case**: Goal planning worksheet

---

### 3. Link to Labeling System

**Question**: "Which label files are assigned to which products?"

**Integration Path**:

1. Product List Report shows LabelFile for each product
2. Cross-reference with label file management system
3. Verify label files exist and are current
4. Ensure all products have assigned labels

**Use Case**: Label file inventory and assignment audit

---

### 4. Link to Serial Production

**Question**: "What cases of product 12301 were produced?"

**Integration Path**:

1. Product List Report shows that product 12301 exists with specs 7.5-8.5 oz
2. Query Serial table for ProductCode = 12301
3. Review Serial records to see all cases produced

**Use Case**: Production case lookup by product

---

### 5. Export for External Systems

**Question**: "How do we share product specifications with external quality management systems?"

**Integration Path**:

1. Generate Product List Report
2. Save `reports\product.txt` or `reports\pproduct.txt`
3. Export to Excel, CSV, or other format
4. Import into quality management or ERP system

**Use Case**: Specification sharing and compliance documentation

---

## 10. Troubleshooting

### Issue: Report Shows Blank or Empty

**Possible Causes**:

1. No products configured in Product table
2. Product table lacks read permissions for current user
3. File system error preventing output file creation
4. Editor widget display issue

**Debugging Steps**:

1. Verify Product table contains records (query database directly)
2. Check user permissions on Product table (read-only needed)
3. Verify `reports\` directory exists and is writable
4. Check s-repviewer.w window handles are valid

---

### Issue: Columns Misaligned or Data Overlapping

**Possible Causes**:

1. Field values exceed allocated column width
2. Non-standard characters or special formatting in descriptions
3. Different font or character width in editor display

**Debugging Steps**:

1. Check ShortDesc fields for excessive length (should be ≤ 8 chars)
2. Verify ProductCode is numeric 5 digits (no non-numeric characters)
3. Check weight values are valid decimals (7.99 format)
4. Ensure LabelFile is ≤ 8 characters

---

### Issue: Print Output Looks Different from Display

**Possible Causes**:

1. Different page sizes (display = 17 lines, print = 66 lines)
2. Different column positions between display and print versions
3. Printer font settings different from display font

**Debugging Steps**:

1. Review `CreateProductReport` (display) vs `ip-printreport` (print) procedures
2. Verify column positions match between both procedures
3. Check printer is using standard fixed-width font
4. Test with different printer drivers if alignment issues persist

---

### Issue: Page Navigation Buttons Hidden But Report Is Long

**Possible Causes**:

1. Report exactly 20 lines or less
2. Logic check (reports ≤ 20 lines hide buttons; > 20 lines show buttons)

**Expected Behavior**:

- Reports with ≤ 20 lines: navigation buttons remain hidden (all content visible in one page)
- Reports with > 20 lines: navigation buttons appear and are enabled

**Resolution**:

- This is expected behavior; no action needed
- If you need to see buttons, add more products to Product table

---

### Issue: Changes to Products Not Appearing in Report

**Possible Causes**:

1. Product changes made but report not regenerated
2. Old cached report still displayed

**Resolution**:

1. Return to Report Menu
2. Click "Product List" button again to regenerate report
3. Verify new data appears in refreshed report

---

## 11. Performance Considerations

### Dataset Size Impact

**Typical Scenario**:

- Product table: 50-200 products per facility
- Average product record: ~100 bytes
- Typical database: 5,000-50,000 bytes total

**Report Performance**:

- Single scan of Product table (no WHERE clause)
- Instant execution (<100 ms) for any facility size
- Formatting and file I/O: <500 ms
- Display loading: <1 second total

**Scalability**:

- No performance degradation even with 1,000+ products
- Linear time complexity: O(n) where n = product count
- No indexing required (sequential scan acceptable)

### Memory & Storage

**Output File Size**:

- Display file (`reports\product.txt`): ~5 KB per 50 products
- Print file (`reports\pproduct.txt`): ~10 KB per 50 products
- Total temporary storage: <100 KB typical

**Editor Widget Memory**:

- ediReport widget can display up to ~1 MB of text
- No memory issues with product lists up to 10,000+ products
- Practical limit: facility has 200-500 products max

### Optimization Opportunities

1. **Caching**: Product list rarely changes; could cache for multi-use scenarios
2. **Filtering**: Could add parameters to show only active/inactive products
3. **Reporting**: Could add subtotals by PackType or WgtUnits
4. **Export**: Could automatically export to CSV for spreadsheet use

---

