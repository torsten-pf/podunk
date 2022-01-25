# podunk

A simple library for creating tabular PDF reports in Python using the excellent ReportLab PDF library (www.reportlab.org). This library was created by Jim Storch (https://code.google.com/archive/p/podunk) and imported and enhanced by jojomaquiling (https://github.com/jojomaquiling/podunk).
This fork adds support for A4 paper, several European date formats and packaging files for PyPI upload. 

Here's an example:

```python
#!/usr/bin/env python

from podunk.project.report import Report
from podunk.widget.table import Table
from podunk.widget.heading import Heading
from podunk.prefab import alignment
from podunk.prefab.formats import format_us_currency
from podunk.prefab.formats import format_two_decimals

table = Table()

col = table.add_column('employee')

col = table.add_column('rate')
col.row.format = format_us_currency
col.row.style.horizontal_alignment = alignment.RIGHT

col = table.add_column('hours')
col.row.format = format_two_decimals
col.row.style.horizontal_alignment = alignment.RIGHT

col = table.add_column('pay')
col.row.format = format_us_currency
col.row.style.horizontal_alignment = alignment.RIGHT

for x in range(10):
    table.add_row(['Smith, John', 10.0, 80.0, 800.0, ])

table.count_column('employee')
table.average_column('rate')
table.sum_column('hours')
table.sum_column('pay')

report = Report('test.pdf')
report.title = 'Payroll for July 18, 2008'
report.author = 'Test Script'
report.add(Heading('A Sample Payroll'))
report.add(table)
report.create()

```

Which creates the test.pdf file.

## Info

* Requires: ReportLab PDF library for Python (www.reportlab.com)
* Tested on: Fedora Linux
* License: BSD (see the example font files for their individual licenses).

## Why Podunk?

I wanted a short, modest name for a short, modest project. PDF stands for Portable Document Format so I scribbled down Podofo and immediately thought of Podunk.

# Main Widgetry

## Report
A Report assembles one or more Headers and Tables into a PDF with a title, date, author, and page numbering on each page.

### Properties:

* pdf_file - name of the file to create
* title
* author
* page_width - in picas
* page_height - in picas
* left_margin - default 54 picas (3/4")
* top_margin - default 72 picas (1")
* right_margin - default 54 picas (3/4")
* bottom_margin - 72 picas (1")
* canvas - the ReportLab Canvas object in case you need 
lower level access

### Methods:

* Add() - Add a report printable object, currently a Header or Table.
* Create() - create the PDF

## Heading

A Heading object is simply a bold, centered Field object (see below) with some vertical padding. It's used much like the HTML tag of the same name -- an optional label for a Table that follows.

Properties:

* field - a Field object
* skip - vertical space to pad above the Heading text, default is 10 picas

## Table

A Table object is where most of the work gets done. You define columns then add rows of data. Columns and rows are printed in the order they are added.

### Properties:

* row_padding - Space between rows, default is 0
* column_padding - Space between columns, default is 4

### Methods:

* add_column(column_name [,width]) - Define a new column. Columns are printed in the order added. Returns the Column object created for tweaking.
* add_row(list) - add a row of data provided as a Python list in the same order as the columns were defined.
* add_dict(dictionary) - add a row of data using a dictionary where the keys match column names. Unlike add_row(), you may omit columns and they will be filled with None.
* average_column(column_name, index) - Fills in the footer with the average of values in the column. None values are skipped.
* count_column(column_name, index) - Fills in the footer with the count of rows in the column. None values are skipped.
* sum_column(column_name, index) - Fills in the footer with the sum of values in the column.
* get_header_field(column_name) - return the header Field for specified column name.
* get_row_field(column_name) - return the row Field for specified column name.
* get_footer_field(column_name) - return the footer Field for specified column name.
* auto_width(canvas) - Shrinks each column to fit the width of the widest element, including headers and footers.
* auto_grow(canvas, width) - Scales the entire table to the given width. Columns are proportional to the width of their elements.
* total_width() - Returns the width of Table; all columns + padding.

# Sub Widgetry

You can ignore these unless you want to tweak or extend the formatting of table elements.

## Field

Most of what you display in Podunk is done via Fields. Fields are made up of the following bits:

* width - Default is 72 (one inch)
* height - Default is 11
* box - See below
* style - See below
* value - Any Python data type. You would have to provide format functions for really odd ones, though.
* format -See below

## Box

A Box object creates a rectangular background with zero to four borders.

### Properties:

* left_border - Width in 1/72 of an inch, None = No line (default for all)
* top_border
* right_border
* bottom_border
* border_color - Color in a triple of RGB in the range 0.0 = 1, e.g. (0,1,0) = green
* border_style - Dash on, dash off dublet, (1,0) = solid
* background_color - Color in a triple of RGB in the range 0.0 = 1, e.g. (.5,.5,.5) = half grey
* line_cap - Type of line endings, see the ReportLab docs for more info.

## Style

A Style object controls the display of text.

### Properties:

* font - A Podunk Font object, default is HELVETICA
* bold - default False
* italic - default False
* size - Font size in picas, default is 7
* horizontal_padding - amount of space from horizontal edges in 1/72 of an inch, default is 2
* vertical_padding - amount of space from vertical edges in 1/72 of an inch, default is 3
* color = Color for the font, default is black (.0,.0,.0)
* horizontal_alignment - default is left
* vertical_alignment - default is bottom

## Format

Formats are simply functions that accept a field's value and return a string. For example, here's a format that converts a float into US currency:

```python
def format_us_currency(value):
    """
    Returns value in monetary format, 2 decimal places, comma separated
    every three digits with a leading dollar sign.
    """
    foo = locale.setlocale(locale.LC_ALL,('en','ascii')) 
    if value == None:
        retval = ''
    else:
        retval = '$ ' + locale.format("%.2f", float(value), True)                  
    return retval
```

## Column

A Column object is a vertical set of data with a header, zero or more rows, and an optional footer.

### Properties:

* name
* width - default is 72 picas (1")
* header - A Field object that controls the look/format of the column name
* row - A Field object that controls the look/format of the rows
* footer - A Field object that controls the look/format of the footer. A footer will not print if the value is None (default).

# Instructions for how to use and embed a TrueType font into your document.

## Introduction

ReportLab cannot directly import a TTF file.  You must provide Adobe AFM ('Adobe Font Metrics') and PFB ('Printer Font Binary') files.  Luckily, you can convert TTF's into AFM and PFB files with a program called FontForge and then import them using Podunk's Font class.

The Font class is extremely simple -- having only four properties that correspond to the name of the typeface in plain (regular), bold, italic, and bold + italic.  These are just string values.  One of the goals of the font class was to be able to refer to built-in PDF fonts and user created ones with exactly the same syntax.

# Details

 * Find your TrueType font.
 * Start up FontForge and open the TTF file.  Some detailed fonts can take several minutes to open, so have patience.  You should see a grid layout of the font's glyphs.
 * Select 'Generate Fonts' from the File menu.
 * Select 'PS Type 1 (Binary)'
 * Click the 'Options' button and insure that 'Output AFM' is checked.
 * Click 'Save' -- Do not try to rename the font, it should be using the exact font name that ReportLab will need during import.  If you get validation errors you may have to un-check 'Validate before Saving' on the same dialog.

You should now have two files with the name of the font that end with '.afm' and '.pfb'.  Now repeat the process with any bold, italic, and bold + italic versions you have for this typeface.

To use these files in Podunk, create an instance of the Font class like so:


```python
from podunk.widget.font import Font

font_info = {
    'plain':'GentiumBasic',         
    'bold':'GentiumBasic-Bold',
    'italic':'GentiumBasic-Italic',
    'bold_italic':'GentiumBasic-BoldItalic',
    'path':'podunk/media/fonts/Gentium',
    }

gentium = Font(font_info)
```

If you don't have all bold/italic variations just repeat the name of the plain font.  

I found that ReportLab will choke on AFM files that have empty comments. If you see an error message like:

```python
ValueError: Header information error in afm font_dir/font_name.afm: line='Comment'
```

Open the afm file in a text editor and delete the line(s) that only contains the word 'Comment' by itself.

See Also:

* http://www.reportlab.org/devfaq.html
* http://fontforge.sourceforge.net/