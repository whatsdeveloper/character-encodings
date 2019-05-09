# Byte order mark (BOM)

## What is a byte-order mark

At the beginning of a page that uses a Unicode character encoding you may find some bytes that represent the Unicode code point `U+FEFF` BYTE ORDER MARK (abbreviated as BOM).

The BOM, when correctly used, is invisible.

Before UTF-8 was introduced in early 1993, the expected way for transferring Unicode text was using 16-bit code units using an encoding called UCS-2 which was later extended to UTF-16. 16-bit code units can be expressed as bytes in two ways: the most significant byte first (big-endian) or the least significant byte first (little-endian). To communicate which byte order was in use, `U+FEFF` (the byte-order mark) was used at the start of the stream as a magic number that is not logically part of the text the stream represents.

The picture below shows the bytes used in a sequence of two-byte characters. Each 2-digit hexadecimal number represents a byte in the stream of text. You can see that the order of the two bytes that represent a single character is reversed for big endian vs. little endian storage. The byte-order mark indicates which order is used, so that applications can immediately decode the content.

![BOM](bom.png)

In the UTF-8 encoding, the presence of the BOM is not essential because, unlike the UTF-16 encodings, there is no alternative sequence of bytes in a character. However, the BOM may still occur in UTF-8 encoded text, either as a by-product of an encoding conversion or because it was added by an editor to flag the content as UTF-8. In this situation, the BOM is often called a **UTF-8 signature**.

## Detecting the BOM

You can find out whether a page contains a BOM at the start or further down in the content by using the [W3C Internationalization Checker](https://validator.w3.org/i18n-checker/). A BOM at the start of the page will be reported in the Information panel. A BOM that is included in the page lower down (typically due to content being added to the page from an external source) will be reported in the Detailed Report section.

You can try looking for a UTF-8 signature in your content in your editor, but if your editor handles the BOM correctly you probably won't be able to see it. With a binary editor capable of displaying the hexadecimal byte values in the file, the UTF-8 signature displays as `EF BB BF`.

## Potential issues with the UTF-8 BOM

What follows are some situations where the byte-order mark has been known to cause problems.

In general, these issues are fading away as people adopt newer versions of browsers and editing tools. It is worth knowing about them if your user base still uses older technology. However, this is not solely about legacy issues.

### PHP includes

At the time this article was written, if you include some external file in a page using PHP and that file starts with a BOM, it may create blank lines.

This is because the BOM is not stripped before inclusion into the page, and acts like a character occupying a line of text. [See an example](https://www.w3.org/International/questions/examples/phpbomtest.php). In the example, a blank line containing the BOM appears above the first item of included text.

You should ensure that the included files do not start with a BOM.

You may also find that the BOM causes problems for an ordinary PHP page. When sending custom HTTP headers the code to set the header must be called before output begins. A BOM at the start of the file causes the page to begin output before the header command is interpreted, and may lead to error messages and other problems in the displayed page.

### Processing with program code

You need to be careful to take the BOM into account in scripts or program code that automatically process files that start with a BOM. For example, when pattern matching at the start of a file that begins with a BOM you need additional code to test for the presence of the BOM and ignore it if found.

The UTF-8 encoding without a BOM has the property that a document which contains only characters from the US-ASCII range is encoded byte-for-byte the same way as the same document encoded using the US-ASCII encoding. Such a document can be processed and understood when encoded either as UTF-8 or as US-ASCII. Adding a BOM inserts additional non-ASCII bytes, so this is no longer true. If you have processes or scripts that assume that the content is comprised of US-ASCII characters only, you will need to avoid the BOM.

### HTTP precedence

Changes introduced with HTML5 mean that the byte-order mark overrides any encoding declaration in the HTTP header when detecting the encoding of an HTML page. This can be very useful when the author of the page cannot control the character encoding setting of the server, or is unaware of its effect, and the server is declaring pages to be in an encoding other than UTF-8. If the BOM has a higher precedence than the HTTP headers, the page should be correctly identified as UTF-8.

At the time of writing, not all browsers do this, so you should not rely on all readers of your page benefitting from this just yet.

In [browsers where the HTTP header still overrides the byte-order mark](https://www.w3.org/International/tests/repository/html5/the-input-byte-stream/results-basics#precedence) and the server is declaring pages to have a non-Unicode character encoding, you are likely to find unexpected characters at the start of the page (such as `ï»¿` in a page labelled in HTTP as ISO 8859-1) as well as problems displaying non-ASCII characters on the page.

### Other issues

If you use applications or scripts in the back end of your site you should check that they are also able to recognize and handle the BOM.

We strongly recommend that you don't change the encoding of a UTF-8 file from a Unicode encoding to a non-Unicode encoding, but if, for some exceptional reason, you do you must ensure that the BOM is removed. If you don't, either the browser will continue to treat your content as UTF-8, or you will see strange characters at the beginning of the page.

## Removing the BOM

If you need to remove the BOM, check whether your editor allows you to specify whether a UTF-8 signature is added or kept while you save the file. Such an editor provides a way of removing the signature by simply reading the file in then saving it out again. For example, in editors such as Notepad++ on Windows and TextWrangler on the Mac, it is possible to select the encoding from a list while using the Save As function. The list has options to save as UTF-8 with or without the BOM. Just choose the option without the BOM and save.
