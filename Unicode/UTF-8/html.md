# Character encodings in HTML

## Modifications to your `php.ini` file

```ini
default_charset = "UTF-8"
```

> Note: You can subsequently use phpinfo() to verify that this has been set properly.

## Modifications to your code

- **Set UTF-8 as the character set for all headers output by your PHP code**

In every PHP output header, specify UTF-8 as the encoding:

```php
header('Content-Type: text/html; charset=utf-8');
```

- **Specify UTF-8 as the encoding type for XML**

```xml
<?xml version="1.0" encoding="UTF-8"?>
```

- **Strip out unsupported characters from XML**

Since not all UTF-8 characters are accepted in an XML document, you’ll need to strip any such characters out from any XML that you generate. A useful function for doing this is the following:

```php
function utf8_for_xml($string)
{
    return preg_replace('/[^\x{0009}\x{000a}\x{000d}\x{0020}-\x{D7FF}\x{E000}-\x{FFFD}]+/u', ' ', $string);
}
```

Here’s how you can use this function in your code:

```php
$safeString = utf8_for_xml($yourUnsafeString);
```

- **Specify UTF-8 as the character set for all HTML content**

For HTML content, specify UTF-8 as the encoding:

```html
<meta charset="UTF-8" />
```

In HTML forms, specify UTF-8 as the encoding:

```html
<form accept-charset="utf-8"></form>
```

- **Specify UTF-8 as the encoding in all calls to** `htmlspecialchars`

e.g.:

```php
htmlspecialchars($str, ENT_NOQUOTES, "UTF-8");
```

> Note: As of PHP 5.6.0, `default_charset` value is used as the default. From PHP 5.4.0, UTF-8 was the default, but prior to PHP 5.4.0, ISO-8859-1 was used as the default. It’s therefore a good idea to always explicitly specify UTF-8 to be safe, even though this argument is technically optional.

Also note that, for UTF-8, `htmlspecialchars` and `htmlentities` can be used interchangeably.

- **Set UTF-8 as the default character set for all MySQL connections**

Specify UTF-8 as the default character set to use when exchanging data with the MySQL database using `mysql_set_charset`:

```php
$link = mysql_connect('localhost', 'user', 'password');
mysql_set_charset('utf8', $link);
```

Note that, as of PHP 5.5.0, `mysql_set_charset` is deprecated, and `mysqli::set_charset` should be used instead:

```php
$mysqli = new mysqli("localhost", "my_user", "my_password", "test");

/* check connection */
if (mysqli_connect_errno()) {
    printf("Connect failed: %s\n", mysqli_connect_error());
    exit();
}

/* change character set to utf8 */
if (!$mysqli->set_charset("utf8")) {
    printf("Error loading character set utf8: %s\n", $mysqli->error);
} else {
    printf("Current character set: %s\n", $mysqli->character_set_name());
}

$mysqli->close();
```

- **Always use UTF-8 compatible versions of string manipulation functions**

There are several PHP functions that will fail, or at least not behave as expected, if the character representation needs more than 1 byte (as UTF-8 does). An example is the `strlen` function that will return the number of bytes rather than the number of characters.

Two options are available for dealing with this:

- The `iconv` functions that are available by default with PHP provide multibyte compatible versions of many of these functions (e.g., `iconv_strlen`, etc.). Remember, though, that the strings you provide to these functions must themselves be properly encoded.
- There is also the `mbstring` extension to PHP (information on enabling and configuring it is available [here](https://www.php.net/manual/en/mbstring.installation.php)). This extension provides a comprehensive set of functions that properly account for multibyte encoding.
