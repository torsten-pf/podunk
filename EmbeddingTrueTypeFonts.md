#Instructions for how to use and embed a TrueType font into your document.

# Introduction #

ReportLab cannot directly import a TTF file.  You must provide Adobe AFM ('Adobe Font Metrics') and PFB ('Printer Font Binary') files.  Luckily, you can convert TTF's into AFM and PFB files with a program called FontForge and then import them using Podunk's Font class.

The Font class is extremely simple -- having only four properties that correspond to the name of the typeface in plain (regular), bold, italic, and bold + italic.  These are just string values.  One of the goals of the font class was to be able to refer to built-in PDF fonts and user created ones with exactly the same syntax.

# Details #

  * Find your TrueType font.
  * Start up FontForge and open the TTF file.  Some detailed fonts can take several minutes to open, so have patience.  You should see a grid layout of the font's glyphs.
  * Select 'Generate Fonts' from the File menu.
  * Select 'PS Type 1 (Binary)'
  * Click the 'Options' button and insure that 'Output AFM' is checked.
  * Click 'Save' -- Do not try to rename the font, it should be using the exact font name that ReportLab will need during import.  If you get validation errors you may have to un-check 'Validate before Saving' on the same dialog.

You should now have two files with the name of the font that end with '.afm' and '.pfb'.  Now repeat the process with any bold, italic, and bold + italic versions you have for this typeface.

To use these files in Podunk, create an instance of the Font class like so:
```
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
```
ValueError: Header information error in afm font_dir/font_name.afm: line='Comment'
```
Open the afm file in a text editor and delete the line(s) that only contains the word 'Comment' by itself.

See Also:
http://www.reportlab.org/devfaq.html
http://fontforge.sourceforge.net/