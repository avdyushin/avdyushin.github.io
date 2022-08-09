---
title: "Using XML and XSLT for code generation"
date: 2020-06-15T07:59:40+02:00
---

XML files can be transformed into different one using XSL templates.
The result of applying template could be another XML, HTML, text or any document.

### Input XML

As an example we will be using XML file contains strings translations:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<strings>
    <string id="login">
        <en>Log in</en>
        <nl>Inloggen</nl>
        <ru>Вход</ru>
    </string>
    <string id="sign_in">
        <en>Sign in</en>
        <de>Anmelden</de>
    </string>
</strings>
```

Each `string` node has `id` attribute contains string identifier.
Using this identifier we are going to build swift `enum` with contains named by identifier.
So later in out code we can use autocompletion and prevent typing errors:

Expected auto-generated `enum`:

```swift
/// Auto-generated
enum Strings {
    static let login = "login"
    static let sign_in = "sign_in"
}
```

### XSLT Template

```xml
<!-- enum.xslt -->
<?xml version="1.0" encoding="UTF-8"?>

<xsl:stylesheet version="1.0"
    xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
    xmlns:fo="http://www.w3.org/1999/XSL/Format">

<xsl:output omit-xml-declaration="yes"/> <!-- 1 -->

<xsl:template match="/"> <!-- 2 -->
/// Auto-generated
enum Strings {
<xsl:for-each select="strings/string"> <!-- 3 -->
    <xsl:text>    static let </xsl:text> <!-- 4 -->
    <xsl:value-of select="@id"/> <!-- 5 -->
    <xsl:text> = "</xsl:text>
    <xsl:value-of select="@id"/>
    <xsl:text>"</xsl:text>
    <xsl:text>&#xa;</xsl:text> <!-- 6 -->
</xsl:for-each>}
</xsl:template>
</xsl:stylesheet>

```

1. To generate plain text we have to ignore default XML header produced by applying template
1. Main rules to apply to root (`/`) element
1. Loops through each `string` node in `strings` node set
1. Writes liter text to the output
1. Extracts values of the `id` attribute
1. Writes new line to the output

Transform XML into text file using `xsltproc` utility:

```sh
$ xsltproc enum.xslt input.xml > strings.swift
```

### Getting distinct values

In order to get a list of all translated languages
we have to iterate over all strings and get distinct nodes only.

To do this we can use `xsl:key` together with `generate-id`:

```xml
<!-- list.xslt -->
<?xml version="1.0" encoding="UTF-8"?>

<xsl:stylesheet version="1.0"
    xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
    xmlns:fo="http://www.w3.org/1999/XSL/Format">

    <xsl:output omit-xml-declaration="yes"/>
    <xsl:key name="lang_id" match="/strings/string/node()" use="name(.)"/> <!-- 1 -->

    <xsl:template match="/">
        <xsl:for-each select="strings/string/node()[generate-id() = generate-id(key('lang_id',name(.))[1])]"> <!-- 2 -->
            <xsl:value-of select="name(.)"/><xsl:text>&#xa;</xsl:text>
        </xsl:for-each>
    </xsl:template>
</xsl:stylesheet>
```

1. Top-level element to define named key as node name
1. Override function to return string that uniquely identifies a node

Applying this template to the input XML will produce a list of unique translated languages:

```sh
$ xsltproc list.xslt input.xml
en
nl
ru
de
```

Next step is to generate `Localized.strings` file for given language.

### Input parameters

Using `xsl:param` we can declare global (or local) parameter.
A `xsl:variable` defines global (or local) variable that will be used as default parameter value:


```xml
<xsl:variable name="defaultLang">
    <xsl:text>en</xsl:text>
</xsl:variable>
<xsl:param name="lang" select="$defaultLang"/>
```

To set parameter from outside we have to pass it as a parameter to `xsltproc`:

```sh
$ xsltproc --stringparam lang ru strings.xslt input.xml
```

### Test conditions

Using `xsl:if` element we can apply template only if `test` condition is true.

* `normalize-space()` trims leading and trailing spaces, so we can filter out strings contains only whitespaces.
* `name()` returns name of the current node
* `text()` returns content of the current node

```xml
<!-- strings.xslt -->
<?xml version="1.0" encoding="UTF-8"?>

<xsl:stylesheet version="1.0"
    xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
    xmlns:fo="http://www.w3.org/1999/XSL/Format">

<xsl:output omit-xml-declaration="yes"/>
<xsl:variable name="defaultLang">
    <xsl:text>en</xsl:text>
</xsl:variable>
<xsl:param name="lang" select="$defaultLang"/>

<xsl:template match="/">
<xsl:text>/// Auto-generated</xsl:text>
<xsl:text>&#xa;</xsl:text>
<xsl:text>/// Language: </xsl:text><xsl:value-of select="$lang"/>
<xsl:text>&#xa;</xsl:text>
<xsl:text>&#xa;</xsl:text>
<xsl:for-each select="strings/string"> <!-- 1 -->
    <xsl:if test="normalize-space(./*[name() = $lang]/text()) != ''"> <!-- 2 -->
        <xsl:text>"</xsl:text>
        <xsl:value-of select="@id"/>
        <xsl:text>" = "</xsl:text>
        <xsl:value-of select="./*[name() = $lang]"/> <!-- 3 -->
        <xsl:text>";</xsl:text>
        <xsl:text>&#xa;</xsl:text>
    </xsl:if>
</xsl:for-each>
</xsl:template>
</xsl:stylesheet>
```

1. Iterate over strings
1. Check if content of node with name $lang (our external parameter) is not empty
1. Get content of the node

### Bash Script

Here is bash script to put all things together:

```bash
!/bin/sh

for lang in $(xsltproc list.xslt strings.xml) # 1
do
    echo "Processing $lang..."
    mkdir -p $lang # 2
    xsltproc --stringparam lang $lang strings.xslt strings.xml > $lang/Localizable.strings #3
done

xsltproc enum.xslt strings.xml > strings.generated.swift #4
```

1. Iterate over languages
1. Make directory if needed
1. Generate strings file
1. Generate `enum` files

### Validating XML

To validate input XML file we can use `xmllint`:

```sh
$ xmllint strings.xml
```

### Formatting XML

In order to sort nodes we can use `xsl:sort` element:

```xml
<!-- sort.xslt -->
<?xml version="1.0" encoding="UTF-8"?>

<xsl:stylesheet version="1.0"
    xmlns:xsl="http://www.w3.org/1999/xsl/Transform">

    <xsl:output method="xml" encoding="UTF-8" indent="yes" omit-xml-declaration="no"/>
    <xsl:strip-space elements="*"/> <!-- 1 -->

    <xsl:template match="@* | node()">
        <xsl:copy> <!-- 2 -->
            <xsl:apply-templates select="@* | node()"/>
        </xsl:copy>
    </xsl:template>

    <xsl:template match="string">
        <xsl:copy>
            <xsl:apply-templates select="@*"/>
            <xsl:apply-templates select="*">
                <xsl:sort select="name(.)" data-type="text" order="ascending"/> <!-- 3 -->
            </xsl:apply-templates>
        </xsl:copy>
    </xsl:template>
</xsl:stylesheet>
```

1. Remove white spaces for all elements
1. Copy current element without child nodes
1. Sort by node name in ascending order

Usage:

```sh
$ xsltproc -o formatted.xml sort.xslt input.xml
```

### Conclusion

Using XSLT it's possible to generate new files based on XML input.
With different templates and parameters we managed to build simple localization platform.

Adding only one more template and step into generation script it could generate translated strings for Android as well.

## Links

- https://github.com/avdyushin/Localizable
- https://www.w3schools.com/xml/xsl_elementref.asp
