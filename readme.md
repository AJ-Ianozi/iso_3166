# ISO 3166-1 country codes for Ada.

This is a full lookup table for the ISO 3166-1 country records in Ada.  In the future, I would like to include iso-3166-2 and iso-3166-3 in this. If that would be useful to you, then read the section below under "Contribute".

**NOTE:** This is currently in pre-release, and may be subject to change.  I'm actively looking for feedback before I publish to Alire.  If this looks good as-is, it'll be published soon, then I'll update the "Installation" section.

## Installation
Coming Soon: Alire integration.

The source files are currently being generated by another app written in Ada, but you don't need to run it in order to use this library, as I've already generated it.  For now, you can just download the ads and adb files under `/src`, or configure your [Alire Pins](https://alire.ada.dev/docs/#work-in-progress-dependency-overrides).

### A note on iso.ads
Right now the [iso_base](https://github.com/ada-iso/iso_base/) package is being provided in /src, it's essentially empty at the moment but there's bigger plans later.  Once iso_base is in alire, that will be removed from here and replaced with a dependency listed in `alire.toml`.

If you want to generate the source files yourself, see below.

## Usage
The library has an enumerated data type called `Country_Keys` compirsed of `C_` followed by a Country Code.  That country key is stored in a `Country` object:
```Ada
type Country_Key is (
   C_AF, --  Afghanistan
   C_AL, --  Albania
   C_DZ, --  Algeria
--    . . .
   C_ZZ  -- Undefined
);
--  The ISO_3166_1 country to be referenced.
type Country is tagged record
   Key : Country_Key := C_ZZ;
end record;
```

The `Country` object has the following member functions / methods:
```Ada
function Name         (This : Country) return String;
function Alpha2       (This : Country) return Alpha2_Code;
function Alpha3       (This : Country) return Alpha3_Code;
function Numeric      (This : Country) return Numeric_Code;
function Country_Code (This : Country) return Ada.Locales.Country_Code;
```
Since these are member functions, you can simply access them via the dot-operator:
```Ada
My_Country : Country := (C_US);
My_Name    : String  := My_Country.Name; -- "United States of America (The)"
```

`Alpha2_Code` and `Alpha3_Code` are subtypes of `String`, and `Numeric_Code` is a subtype of `Natural`.  Each are restricted to the only possible values defined  by the standard and `"ZZ"`, `"ZZZ"`, and `0` respectively for an Undefined country:   
```Ada
subtype Alpha2_Code is String (1 .. 2)
   with Dynamic_Predicate => Alpha2_Code in
      "AF" | "AL" | "DZ" | ... | "ZZ";
subtype Alpha3_Code is String (1 .. 3)
   with Dynamic_Predicate => Alpha3_Code in
      "AFG" | "ALB" | "DZA" | ... | "ZZZ";
subtype Numeric_Code is Natural
   with Dynamic_Predicate => Numeric_Code in
      004 | 008 | 012 | ... | 0; 
```

There are several functions that return this object, depending on what item you're using to look it up:
```Ada
with ISO.Countries; use ISO.Countries;
with Ada.Locales;
declare
   --  All of these will return the USA country.
   USA : constant Country := From_Alpha2 ("US");   --  Country Code
   AUS : constant Country := From_Alpha3 ("AUS");  --  Country Code3
   BIH : constant Country := From_Numeric (070);   --  Numeric
   ATA : constant Country := From_Numeric ("010"); --  Numeric String
   --  Get more information from Ada.Locales country code.
   Local_Country : Country := From_Country_Code (Ada.Locales.Country¨);
begin
   Assert (USA.Name = "United States of America (the)");
   Assert (AUS.Name = "Australia");
   Assert (BIH.Name = "Bosnia and Herzegovina");
   Assert (ATA.Name = "Antarctica");
end;
``` 
There's also an array that can contain all of the countries.  It must be initated with the function `Init_Countries`.  The index is an item of `Country_Key`: 
```Ada
with ISO.Countries; use ISO.Countries;
with Ada.Text_IO;   use Ada.Text_IO;
declare
   A : constant All_Countries := Init_Countries;
begin
   Put_Line (A (C_US).Name);
   for X of A loop
      Put_Line (X.Name);
      Put_Line (X.Alpha2);
   end loop;
end;
```

You can also read the [full API documentation](https://ada-iso.github.io/docs/countries/index.html) which has been generated with [ROBODoc](https://github.com/gumpu/ROBODoc).

## Generating an update
1. Create the CSV
   1. Visit the [Country Code Search](https://www.iso.org/obp/ui/#search/code/)
   2. Be sure to set the page length to 300 to catch everything.
   3. Paste the table into a CSV
   4. **REMOVE** the French Short Name, column B.
   5. Verify the CSV Has the following headers in this exact order: `English short name,Alpha-2 code,Alpha-3 code,Numeric`
   6. Save the CSV at the root level of the `generate` folder.
2. Build the generate app
   1. `cd generate`
   2. `alr build`
3. Run the generate app
   1. `bin/generate "countries.csv"`
   2. The source files will be stored in the `generate/output` directory.
4. Copy the source files to `iso_countries/src` (not `generate/src`!) folder.

## Sources
https://www.iso.org/obp/ui/#search/code/

## Contribute
If someone would like to give me 300 CHF (however much that is in USD) every time a new version of the [Country Codes Collection](https://www.iso.org/publication/PUB500001.html) is made available, I'll be happy to start using XML dumps like in the iso4217 library.

If the Country Codes Collection contains iso-3166-2 and iso-3166-3, I can start including those to build a more complete list.
NOTE: I probably won't be able to share the XML files if this happens, but I can still provide the generator once written.