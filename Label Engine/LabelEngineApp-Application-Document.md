# Label Engine Application — Application Document

---

## 1. Purpose of the Application

- **Product identity**: The application is named **LabelEngine** (AssemblyTitle, AssemblyProduct). The executable and project use the name **LabelEngineApp** (AssemblyName, RootNamespace in the .vbproj). The company is **Computerway Food Systems** (AssemblyCompany). Copyright is stated as 2020 (AssemblyCopyright).
- **Startup**: Entry point is `Sub Main` in `ModMain.vb`. It enables Visual Styles, creates the main form (`FrmMain`), shows a splash screen (`FrmSplash`), calls `mainfrm.LoadMainForm()` to load the main form, closes the splash, then runs the application with `Application.Run(mainfrm)`.
- **Splash**: `FrmSplash` displays the application title, company name, and version (Major.Minor.Revision from `My.Application.Info`).
- **Functional role**: The application is a **label design and print** tool. It allows users to create, edit, save, and print label layouts. Labels are stored in `.lbl` files. The application supports text, database fields, barcodes, images, and shapes on a design surface; configures a label printer (serial or other); manages product/default data for label fields; and can download images to the printer and run communication tests. Status bar text on load is "Computerway Label Design...", and the main form title is "LabelEngine" or "LabelEngine " plus the opened file path.

---

## 2. Themes and Concepts

### 2.1 Label design and layout

- **Label definition**: A label is represented in memory by `LabelArray()` of type `LabelDefinition` (in `ModLabelDesign.vb`). Each element has: `Object_Renamed` (e.g. "DB", "Barcode", "Image", "Shape"), `PosTop`, `PosLeft`, `ObjSize` (width/height), `Orientation`, `Data`, `Font`, `WidthMult`, `HeightMult`, `Misc`, `Misc2`, `DispValue`.
- **Drawing area**: The main form uses virtual picture controls (`PictureBoxVirtual`) for hold, view, and label display. Rulers and a resizable drawing area are initialized. Label dimensions are set via default width/height (e.g. DefLblWth "5.00", DefLblHgt "6.00") and can be changed in the Label Format dialog.
- **Object types**: Supported object types include: text/database field ("DB"), barcode ("Barcode"), image ("Image"), and shape ("X" with sub-types such as box "B" and line "b"). Orientation is stored in tenths of degrees (0, 1800, 2700, 4500, etc.).
- **Multi-label**: Up to 5 label variants can be stored in `MultLabelArray(5, 200)` and `MultLabelSize(5)`. The file format uses a line "E" to separate multiple labels; when the fifth "E" is encountered during load, multi-label loading stops.
- **Twips**: Positions and sizes use a "twips per unit" conversion (`TwipsPerPU`) for mapping between design units and pixel coordinates.

### 2.2 Label file format (.lbl)

- **Storage**: Labels are text files, one line per element or control. Lines are read with `LineInput` and written with `PrintLine`.
- **Control/header lines** (from `Load_Label` and `Save_Label`):
  - `Chr(2) & "e"` — STXe (enable flag).
  - `Chr(2) & "J"` — STXJ (pause-related).
  - `Chr(2) & "L"` — label start marker.
  - `"/Label Size: #.##x#.##"` — label width and length (e.g. "5.00x6.00"); when loading, width must be &gt; 0 and ≤ 8.5, height &gt; 0 and ≤ 20 for automatic resize.
  - `"/Modify Date: " & Now` — modification date.
  - `"C" & ColOffset` — column offset (if IncludeColOffset).
  - `"R" & RowOffset` — row offset (if IncludeRowOffset).
  - `"P"`, `"p"`, `"S"` — print speed, backup speed, slew speed (if included); value encoded as `Chr((speed/0.5)+63)`.
  - `"D" & HgtDotSize & WthDotSize` — dot size.
  - `"H" & HeatSetting` — heat setting (if included).
  - `"z"` — zero conversion flag.
  - `"Q####"` — quantity to print.
  - `"E"` — separator for multi-label (next label).
- **Object lines**: Lines longer than 15 characters with non-blank content after position 15 and not starting with "/" are validated and parsed by `Validate_Line`. Format includes: rotation digit (1–4), item type, width/height multipliers (digit or A–Z for &gt;9), barcode height (3 chars), row position (4 digits), column position (4 digits), then type-specific data (e.g. shape type and dimensions, image name, barcode type and data, DB field identifier).
- **Autosave**: Default save path can be `Autosave.lbl` in the application directory. On open, if no file is chosen, the application may load `autosave.lbl` and treat the label as invalid if `LabelArraySize = 0` after load.

### 2.3 Fonts and typography

- **Font table**: Font definitions are stored in the local database table **Fonts** and loaded into `FontTableArray`. Each entry has: FontID, FontName, FontSize, FontHSize, FontVSize, FontUpperCase, FontWeight, FontVAdjust, FontHPercent, FontVPercent, FontWidthMult, FontHeightMult.
- **PC fonts**: The application also uses the system’s installed fonts (`InstalledFontCollection`); when FontID is "9", a PC font name can be stored in `Misc`. Font width/height multipliers are stored per label object.

### 2.4 Barcodes

- **Barcode catalog**: `modBarcodes.vb` defines `BarcodeArray` (size 24) of type `BarcodeSetup`. Each entry has: BarcodeID (single letter A–X), BarcodeName, MinLength, MaxLength, IsNumeric, IsUpperCase, IsLowerCase, SpecialCharacters, ValidWidth, ValidHeight, HumanReadable, ImageName, Notes, RatioUnlocked, DefaultHeight, FixedWidth (optional).
- **Barcode types** present in code include: Code 3 of 9 (A), UPC-A (B), UPC-E (C), Interleaved 2 of 5 (D), Code 128 (E), EAN-13 (F), EAN-8 (G), HBIC (H), Codabar (I), I2of5 with modulo 10 (J), Plessey (K), I2of5 with bearer bars (L), 2-digit UPC Addendum (M), 5-digit UPC Addendum (N), Code 93 (O), Postnet (P), UCC/EAN Code 128 (Q), and others through index 24. Each type has defined length rules, character sets, and valid width/height ratios.
- **Designer**: The main form’s barcode combo is populated via `Setup_Barcode(cboBarcodeType)`.

### 2.5 Database and configuration

- **Local database**: The application uses a Microsoft Jet (Access) database: `Data\LabelEngine.mdb`, connection string in `ModLabelDesign.vb`: `Provider=Microsoft.Jet.OLEDB.4.0;Data Source=...\Data\LabelEngine.mdb`.
- **Tables referenced in code**: **Fonts**, **Configs**, **LabelFields**, **FieldComponents**, **CrossRef**, **Product**, **ProductText**, **MasterText**. Configs are filtered to exclude OptionName in ('DBConnection', 'ProdDBConnMode', 'SQLUser', 'SQLPW', 'SQLDBName', 'SQLIP') in the config editor; DB connection is edited separately.
- **Config options** (OptionName / OptionValue) used in code include: SearchLocalOrPathFirst, PathToImages, DefLabelWidth, DefLabelHeight, PrinterType, PrinterPort, PrinterSettings, PrinterDevice, CAndG, AutoAdjust, IncludeColOffset, IncludeRowOffset, IncludePrintSpeed, IncludeBackupSpeed, IncludeQtyToPrint, IncludeHeatSetting, IncludeSlewSpeed, DBConnection (with OptionValue2 for connection string).
- **FieldsDB and LabelType**: Defaults on load are `FieldsDB = "Inven"` and `LabelType = "Serial"`. Label fields are queried by `DB` and `LabelType` (e.g. `LabelFields where DB = FieldsDB And LabelType = LabelType`). CrossRef is queried by DB, LabelType, and LabelField (e.g. Order, Lot, Customer, User, Shift, Scale, Plant).
- **Product and product text**: Product data is loaded from **Product** and **ProductText**; **MasterText** is used for subclasses of product text. Product setup variables in `ModLabelDesign` include PlantID, ProdCode, ScaleNum, Shift, UserName, Description1/2, Short1/2/3, Customer, OrderNum, LotNum, RoundOrTruncate, RoundDigits, WgtUnits, NetWgt, LabelWgt, BoxTare, MoistureTare, offsets, Price, LabelFile, and arrays ProdText(99), UserDef(16), GraphicText(10).

### 2.6 Printer and output

- **Printer types**: Config supports `PrinterType = "Serial"` or `"Other"`. Serial uses PrinterPort and PrinterSettings (e.g. baud, parity, data bits); Other uses PrinterDevice (e.g. Windows printer name).
- **Serial settings**: Stored as a comma-separated string (e.g. baud, parity, data bits). Baud values in UI include 1200, 2400, 4800, 9600, 14400, 19200, 38400. Parity: E, O, N.
- **Control characters**: STXc, STXe, STXf, STXJ, STXO, STXz and UseCG (CAndG) are used for printer control. ColOffset, RowOffset, PrintSpeed, BackupSpeed, SlewSpeed, HeatSetting, QtyToPrint, ZeroConversion, HgtDotSize, WthDotSize are written to the label file when the corresponding Include* option is true.
- **Print path**: Printing uses Windows printer APIs (e.g. `WritePrinter`, `EndPagePrinter`, `EndDocPrinter`). Label content is built from the label array and sent to the selected printer.

### 2.7 Images

- **Image directory**: Configurable; default is `Images\` under the application directory. If Config `SearchLocalOrPathFirst` is "PATH", `PathToImages` from Config is used as ImageDir.
- **Image format**: Only 1-bit (monochrome) BMP files are treated as valid for the image download feature; `ListAvailableImages()` checks `biBitCount = 1`.
- **Image objects on labels**: Image elements store a path or name; when saving, the path is stripped to the filename and optional bracketed text placeholder (e.g. [TEXTxx]) is preserved.

### 2.8 Data source / field mapping

- **DataSourceID**: In `ModLabelDesign`, field IDs are mapped to categories by the first few characters (e.g. "BAR"→Barcode, "LOT"→Lot, "ORD"→Order Number, "PRO"→Product Code, "SER"→Serial Number, "TEX"→Text, "UNI"→Unique 8, "USE" with "USERBAR-*"→Userbar else User Name). Otherwise "Text".
- **LabelFields**: Fields available for the current DB and LabelType are loaded from **LabelFields**; **FieldComponents** is used for component details. Field description is used to match dropdowns (e.g. cboDataSource, cboDBFields).

### 2.9 Updates

- **Update folder**: If the application finds a folder `Updates\` under its directory, it runs `InstallUpdate()`.
- **Update files**: The procedure looks for `Fonts.upd`, `LabelFields.upd`, `FieldCom.upd`, or `Configs.upd`. If any exist, it prompts "Updates have been found. Do you want to install them now?" and, if the user confirms, applies updates to the corresponding tables (Fonts, LabelFields, FieldComponents, Configs) and then deletes the .upd files.

### 2.10 Paths and directories

- **Standard paths** (with or without trailing backslash on `My.Application.Info.DirectoryPath`): LblDesign.ini, Logs\, Labels\, Images\. Logs, Labels, and Images are created if missing.
- **Import**: CSV import path is hard-coded as `C:\Program Files\LabelEngine\Import.csv`. The format expects lines with: vObj, vFont, vHgt, vWth, vOrient, vPosLeft, vPostop, vMisc, vObjHgt, vObjWth, vData; only lines with vObj = "DB" are imported as label objects.

---

## 3. Business Rules (from source only)

### 3.1 Label format (FrmLabelFormat)

- **Label height**: On leave of the height field, value must be numeric and between 1 and 20 (inclusive). Otherwise a message is shown: "The height must be a number between 1 And 20." and focus returns to the field. Display format is "0.00".
- **Label width**: On leave of the width field, value must be numeric and between 1 and 8.5 (inclusive). Otherwise a message is shown: "The width must be a number between 1 And 8.5." and focus returns to the field. Display format is "0.00".
- **Defaults when empty**: If ColOffset, RowOffset, or PrintSpeed are empty on load, defaults are set: ColOffset "0", RowOffset "0", Width "5.00", Height "6.00", PrintSpeed " 6.5", Backup " 4.0", Slew " 6.5", Qty "1", Heat "10", Zero conversion option 1, AutoAdjust unchecked, Pause option 1, CAndG option 0, and all Include* checkboxes checked.

### 3.2 Product setup (FrmProductSetup)

- **Numeric fields**: On leave, txtNetWgt, txtLabelWgt, txtBoxTare, txtMoistureTare are forced to numeric; if not numeric they are set to "0". Round/Truncate and RoundDigits are then applied to format the corresponding display labels.
- **Price**: On leave of txtPrice, value must be numeric; otherwise the message "Price must be set to a numeric value." is shown and focus returns to the field.
- **Round/Truncate**: Displayed weight values are computed from the text fields using cboRoundTruncate ("Round" or "Truncate") and cboRoundDigits. Round uses `Math.Round` with the chosen digit count; Truncate uses substring from the decimal point.
- **Offset buttons**: Offset1, Offset2, VOffset are adjusted by ±1 via buttons; the corresponding label shows the date as `DateTime.Today.AddDays(Val(txtOffset*))` formatted as "MM/dd/yyyy".

### 3.3 Userbar (FrmUserbar)

- **Userbar length**: On leave of txtUBLength, value must be numeric and must not contain a decimal point. Otherwise the message "The Userbar Length must be a valid integer number." is shown, value is set to 1, and focus returns to the field.
- **Userbar list**: The combo lists characters A–Z (Chr(64 + ArrayLoc) for ArrayLoc 1 to 26).

### 3.4 Configs (FrmConfigs)

- **Load**: Only rows where OptionName is not 'DBConnection', 'ProdDBConnMode', 'SQLUser', 'SQLPW', 'SQLDBName', 'SQLIP' are loaded, ordered by OptionName. If no rows remain, the message "There are no configs to setup." is shown and the form closes.
- **Save**: On form closing, the grid’s changes are written back via the adapter’s UpdateCommand and Update(tblConf).

### 3.5 Database connection (FrmDBConnectionSetup)

- **Load**: Only Config rows with OptionName = 'DBConnection' are loaded. OptionValue is shown as connection name, OptionValue2 as connection string. Buttons Previous/Next/Save/Cancel/Add/Delete state is set from row count.
- **Connection string**: If OptionValue2 is empty, username, password, IP, port are cleared and Progress DB type is selected. If OptionValue2 starts with "http", Progress AppServer is selected and username/password are disabled.

### 3.6 Label file (FrmMain)

- **Invalid label**: After loading a label file (e.g. autosave.lbl), if `LabelArraySize = 0`, the message "Label File is invalid. It contains nothing to print." is shown and the form title is set to "LabelEngine" (open is effectively aborted).
- **Label size on load**: When parsing "/Label Size: #.##x#.##", SizeX must be &gt; 0 and ≤ 8.5 and SizeY &gt; 0 and ≤ 20 for the form to apply the size and resize automatically. If the opened file’s size differs from the default and no resize was yet applied, the user is asked "Do you wish to open the label in the default label size?" (Yes/No).
- **File dialogs**: Open and Save use filter "Labels (*.lbl)|*.lbl"; InitialDirectory is LabelDir when appropriate.
- **Import**: If `C:\Program Files\LabelEngine\Import.csv` does not exist, the message "Import file does not exist !!" is shown.

### 3.7 Image download (FrmDownloadImages)

- **Images directory**: If ImageDir is not found or the path is empty, the message shows that the directory was not found and the screen closes.
- **No images**: If no files are found in ImageDir or no valid (1-bit BMP) images are found, messages state that there are no images or no valid images (valid = black and white bitmaps).
- **Download**: Before starting download, there must be at least one image selected; otherwise "There are no images selected to download to the printer." is shown. If memory module is "N/A", "There are no memory modules available." is shown. For USB, if no printer is selected, "Please select a printer to download the images from the drop-down list." is shown. If communication fails (res = 0), "LabelEngine is unable to communicate with printer through the USB Port." is shown.
- **Comm port**: If the selected comm port appears in use or invalid, the message states that the port may be in use by another program or is not a valid comm port.

### 3.8 Load_Label / Validate_Line

- **Line validation**: In `Validate_Line`, the first character of an object line must be numeric (rotation). Width/height multipliers at positions 2–3 that are A–Z are converted with Asc(...)-55; if Val is 0 they are set to 1. Row and column positions (4-digit numeric substrings) are required at the expected positions; failure to parse causes the function to exit (ValidLine false).
- **Multi-label cap**: When loading, after the fifth "E" line, `Check_MultLabel()` is run and the subroutine exits (no more than 5 labels in a multi-label file).

### 3.9 Save_Label

- **TempHold bounds**: When computing row/column position for saving, if TempHold &lt; 0 it is set to 1 before formatting.
- **Width/height multiplier encoding**: For barcodes, if WidthMult or HeightMult &gt; 9, the value is encoded as Chr(55 + Floor(value)) (e.g. 10→'A').

### 3.10 ListAvailableImages (ModLabelDesign)

- **Valid images**: Only files in ImageDir with extension .bmp and with `biBitCount = 1` in the BMP info header are added to the list of available images.

### 3.11 InstallUpdate (FrmMain)

- **Prompt**: If any of Fonts.upd, LabelFields.upd, FieldCom.upd, Configs.upd exist, the user is asked "Updates have been found. Do you want to install them now?" If the user chooses No, "Update canceled by user." is shown and the procedure exits.
- **Apply and delete**: After applying updates to the corresponding tables, the .upd files are deleted.

---

## 4. Menu and Form Structure (from code)

- **File**: New, New Multi, Printer Setup, Print, Print to Printer, Open, Save, Save As, Save Image, High Image, Exit.
- **Edit**: Undo, Redo.
- **Setup**: Control Setup, Label Format, Config Editor.
- **Product**: Product Ref, Q Product Retrieve, Prod Text Upload, DB Connection Maint, SQL Product Retrieve, Userbar, Download Images, Comm Test, Memory, Field Editor, Fonts.
- **Tools**: Image Editor, Label Format Text Editor, Copy Images to Server, Import.
- **Help**: About.

Forms referenced include: FrmMain, FrmSplash, FrmLabelFormat, FrmLabelFormatEditor, FrmProductSetup, FrmConfigs, FrmPrinter, FrmPrinterControl, FrmUserbar, FrmDBConnectionSetup, FrmConnectionEditor, FrmDownloadImages, FrmImageDownloadCommSettings, FrmImageEditor, FrmCopyImages, FrmTextFields, FrmTextFields2, FrmAbout, FrmCommTest, FrmMemoryCheck, FrmMemTest, FrmReboot, FrmPleaseWait, FrmCrossRefEditor, FrmProductRetrieve, FrmImagePreview, FrmFontEditor.

---

## 5. Technology and Versions

- **Framework**: .NET Framework 4.7.2 (TargetFrameworkVersion v4.7.2).
- **Language**: VB.NET (Option Explicit On, Option Compare Binary, Option Strict Off, Option Infer On).
- **UI**: Windows Forms; Visual Styles enabled.
- **Data**: OleDb (Microsoft.Jet.OLEDB.4.0) for LabelEngine.mdb.
- **Assembly version**: 2.7.0.0 (from AssemblyVersion and ApplicationVersion in the project and from FrmSplash display).

---


