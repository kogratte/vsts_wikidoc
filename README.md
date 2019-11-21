# wikidoc

This python script allows to create nice looking PDF files from a VSTS wiki, which can be used as offline/printable documentation. It is a wrapper for pandoc and wkhtmltopdf.

## Prerequisite ##

First, you need to install used binaries.

- wkhtmltopdf can be found [here](https://wkhtmltopdf.org/)
- pandonc can be found [here](https://pandoc.org/)

## Concept ##

All the information needed to configure wkhtmltopdf are stored in the wiki home file as html comments, so no additional files/options are needed to generate the PDF. If the wiki home file is setup as needed, clone the wiki and run this script as follows, to generate the PDF:

```
git clone https://github.com/<user>/<repository>.wiki.git
/path/to/wikidoc.py <path-to-wkhtmltopdf> <path-to-github-wiki-repository>
```

Example:
```
wikidoc.py "C:\Program Files\wkhtmltopdf\bin\wkhtmltopdf.exe" "D:\Git Repos\myProject.wiki"
```


For this to work, the github wiki must be setup as _github markdown_, which allows html tags and thus html comments. The wiki markdown files are converted via pandoc to html and are joined to one large html file, which is then converted to PDF using wkhtmltopdf. The generated html file is deleted, to be able to look up the html id tags created by pandoc, which may be used to style the PDF. Writing the wiki in pure html in the first place makes things a lot easier.

If you don't have a file named Home.md, the provided DefaultHome.md will be used.

If you want to customize the pdf layout, you can copy the DefaultHome.md to <path-to-github-wiki-repository>/Home.md.

If you're missing of capabilities working on the generated pdf, please feel free to comment the last three lines of wikidoc.py. You're gonna be able to work on the generated html, wich can be more convenient.

## How it works ##

The provided DefaultHome.md contains all possible wikidoc html comments, but you'll find the optionnal ones below:

```
<!-- WIKIDOC COVER
<div class='covertitle'><b>Example</b> Documentation<br><span class='generated'>generated from github wiki: ###_WIKIDOC_GENDATE_###</span></div>
WIKIDOC COVER -->

<!-- WIKIDOC TOCXSL
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet version="2.0"
                xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
                xmlns:outline="http://wkhtmltopdf.org/outline"
                xmlns="http://www.w3.org/1999/xhtml">
  <xsl:output doctype-public="-//W3C//DTD XHTML 1.0 Strict//EN"
              doctype-system="http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd"
              indent="yes" />
  <xsl:template match="outline:outline">
    <html>
      <head>
        <title>Contents</title>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
        <style>
          h1 {
            text-align: left;
            font-size: 26.6px;
            font-family: verdana;
          }
          div {border-bottom: 1px dashed rgb(200,200,200);}
          span {float: right;}
          li {list-style: none;}
          ul {
            font-size: 13px;
            font-family: verdana;
          }
          ul ul {font-size: 85%; }
          ul {padding-left: 0em; }
          ul ul {padding-left: 1em;}
          a {text-decoration:none; color: black;}
          ul.toplevel li { margin-bottom: 1em;  }
          ul.sublevels li { margin-bottom: 0em;  }
          ul li:last-child { margin-bottom: 1em; }
        </style>
      </head>
      <body>
        <h1>Contents</h1>
        <ul class="toplevel"><xsl:apply-templates select="outline:item/outline:item"/></ul>
      </body>
    </html>
  </xsl:template>
  <xsl:template match="outline:item">
     <li>
      <xsl:if test="(@title!='') and (@title!='Contents')">
        <div>
          <a>
            <xsl:if test="@link">
              <xsl:attribute name="href"><xsl:value-of select="@link"/></xsl:attribute>
            </xsl:if>
            <xsl:if test="@backLink">
              <xsl:attribute name="name"><xsl:value-of select="@backLink"/></xsl:attribute>
            </xsl:if>
            <xsl:value-of select="@title" /> 
          </a>
          <span> <xsl:value-of select="@page" /> </span>
        </div>
      </xsl:if>
      <ul class="sublevels">
        <xsl:comment>added to prevent self-closing tags in QtXmlPatterns</xsl:comment>
        <xsl:apply-templates select="outline:item"/>
      </ul>
    </li>
  </xsl:template>
</xsl:stylesheet>
WIKIDOC TOCXSL -->

<!-- end of wikidoc config section -->

<!-- WIKIDOC PDFONLY
<h1>###_WIKIDOC_TITLE_###</h1>
WIKIDOC PDFONLY -->

Only this text is shown on the wiki homepage. Since the title of this wiki file (Home) is not
part of the file itself (the github wiki generates it from the filename), it would be missing in the
pdf. To overcome this, the WIKIDOC PDFONLY comment can be used: The content of this comment
is actually rendered in the PDF and the placeholder ###_WIKIDOC_TITLE_### is replaced by a title
generated from the filename in the same way the github wiki is doing it. However, you are free to
choose a different name for the pdf and/or a different header-level (h2, h3 ...), or leave it out completely.

The WIKIDOC PDFONLY comment can be used in all wiki files, not just the wiki home file.
```

## Comments usable in wiki home ##

* WIKIDOC CONFIG: This required wikidoc comment contains a list of parameter definitions. All except "filename" will be directly send as parameters to wkhtmltox (see [documentation](http://wkhtmltopdf.org/usage/wkhtmltopdf.txt) of wkhtmltox for a list of possible options). The provided parameters are not verified by wikidoc. If the filename is missing, the default "wikidoc.pdf" will be used.

* WIKIDOC HTMLHEAD and WIKIDOC HTMLFOOT: The wiki markdown files are converted to html with pandoc, joined in the same order as listed on the github wiki and are put between the HTMLHEAD and HTMLFOOT segements provided by these two required  WIKIDOC comments. The HTMLHEAD may also contain a CSS STYLE section to style the PDF.

* WIKIDOC COVER: This is an optional comment. If present, it will create a cover page which is special to wkhtmltopdf as it will never have any header or footer. The COVER comment may only contain html (no markdown allowed!) and supports all available placeholder substitutions (for example ###_WIKIDOC_GENDATE_###).

* WIKIDOC TOCXSL: This is an optional comment. If present, a table of contents will be added to the PDF, after the cover and before the actual document. The TOC is completely defined by the provided XSL. Any provided wkhtmltopdf parameter for the toc section will be ignored.

## Comments usable in all wiki files ##

* WIKIDOC PDFONLY: The html content (no markdown allowed!) of this comment will be redered in the PDF. The original purpose was to provide a flexibel solution to add a header to each wiki file (see botttom of example `home.md`). However, it can be used to provide any html content for the PDF, which might not be supported by the github wiki markdown. For example complex tables: A simple table could be designed for the wiki, which has a CSS class which hides it in the PDF and the PDFONLY contains a more complex table for the PDF. 

If the PDFONLY section is given a name as in the following example, the section will also be generated as a PNG, which can be used in the wiki, to overcome limitations of the github markdown. The PNG will get its name from the section name and ist stored inside the wiki repository in a folder called `generated-images`. That folder must exist.

```
<!-- WIKIDOC PDFONLY examplewithname
<table CELLPADDING="3" 
...
</table>
WIKIDOC PDFONLY -->
```
