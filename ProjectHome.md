A simple library for creating tabular PDF reports in Python using the excellent ReportLab PDF library (www.reportlab.org).  Here's an example:

```
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

Which creates:  http://podunk.googlecode.com/files/test.pdf

## Info ##

  * Requires:  ReportLab PDF library for Python (www.reportlab.com)
  * Tested on: Fedora Linux
  * License:  BSD (see the example font files for their individual licenses).


## Why Podunk? ##

I wanted a short, modest name for a short, modest project.  PDF stands for Portable Document Format so I scribbled down Podofo and immediately thought of Podunk.


## Main Widgetry ##


### Report ###

A Report assembles one or more Headers and Tables into a PDF with a title, date, author, and page numbering on each page.

Properties:
  * pdf\_file - name of the file to create
  * title
  * author
  * page\_width - in picas
  * page\_height - in picas
  * left\_margin - default 54 picas (3/4")
  * top\_margin - default 72 picas (1")
  * right\_margin - default 54 picas (3/4")
  * bottom\_margin - 72 picas (1")
  * canvas - the ReportLab Canvas object in case you need lower level access

Methods:
  * Add() - Add a report printable object, currently a Header or Table.
  * Create() - create the PDF

### Heading ###

A Heading object is simply a bold, centered Field object (see below) with some vertical padding.  It's used much like the HTML tag of the same name -- an optional label for a Table that follows.

Properties:
  * field - a Field object
  * skip - vertical space to pad above the Heading text, default is 10 picas

### Table ###

A Table object is where most of the work gets done.  You define columns then add rows of data.  Columns and rows are printed in the order they are added.

Properties:
  * row\_padding - Space between rows, default is 0
  * column\_padding - Space between columns, default is 4

Methods:
  * add\_column(column\_name [,width]) - Define a new column.  Columns are printed in the order added.  Returns the Column object created for tweaking.

  * add\_row(list) - add a row of data provided as a Python list in the same order as the columns were defined.
  * add\_dict(dictionary) - add a row of data using a dictionary where the keys match column names.  Unlike add\_row(), you may omit columns and they will be filled with None.

  * average\_column(column\_name) - Fills in the footer with the average of values in the column.  None values are skipped.
  * count\_column(column\_name) - Fills in the footer with the count of rows in the column. None values are skipped.
  * sum\_column(column\_name) - Fills in the footer with the sum of values in the column.

  * get\_header\_field(column\_name) - return the header Field for specified column name.
  * get\_row\_field(column\_name) - return the row Field for specified column name.
  * get\_footer\_field(column\_name) - return the footer Field for specified column name.

  * auto\_width(canvas) - Shrinks each column to fit the width of the widest element, including headers and footers.
  * auto\_grow(canvas, width) - Scales the entire table to the given width. Columns are proportional to the width of their elements.
  * total\_width() - Returns the width of Table; all columns + padding.


## Sub Widgetry ##

You can ignore these unless you want to tweak or extend the formatting of table elements.


### Field ###

Most of what you display in Podunk is done via Fields.  Fields are made up of the following bits:

  * width - Default is 72 (one inch)
  * height - Default is 11
  * box - See below
  * style - See below
  * value - Any Python data type.  You would have to provide format functions for really odd ones, though.
  * format -See below


### Box ###

A Box object creates a rectangular background with zero to four borders.

Properties:
  * left\_border - Width in 1/72 of an inch, None = No line (default for all)
  * top\_border
  * right\_border
  * bottom\_border
  * border\_color - Color in a triple of RGB in the range 0.0 = 1, e.g. (0,1,0) = green
  * border\_style - Dash on, dash off dublet, (1,0) = solid
  * background\_color - Color in a triple of RGB in the range 0.0 = 1, e.g. (.5,.5,.5) = half grey
  * line\_cap - Type of line endings, see the ReportLab docs for more info.


### Style ###

A Style object controls the display of text.

Properties:
  * font - A Podunk Font object, default is HELVETICA
  * bold - default False
  * italic - default False
  * size - Font size in picas, default is 7
  * horizontal\_padding - amount of space from horizontal edges in 1/72 of an inch, default is 2
  * vertical\_padding - amount of space from vertical edges in 1/72 of an inch, default is 3
  * color = Color for the font, default is black (.0,.0,.0)
  * horizontal\_alignment - default is left
  * vertical\_alignment - default is bottom

### Format ###

Formats are simply functions that accept a field's value and return a string.  For example, here's a format that converts a float into US currency:

```
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

### Column ###

A Column object is a vertical set of data with a header, zero or more rows, and an optional footer.

Properties:
  * name
  * width - default is 72 picas (1")
  * header - A Field object that controls the look/format of the column name
  * row - A Field object that controls the look/format of the rows
  * footer - A Field object that controls the look/format of the footer.  A footer will not print if the value is None (default).

