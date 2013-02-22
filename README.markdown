# Getting started

GUMP is a standalone PHP input validation and filtering class.

1. Download GUMP
2. Unzip it and copy the directory into your PHP project directory.

Include it in your project:

```php
require "gump.class.php";
```

Methods available:

```php
validation_rules(array $rules); // Get or set the validation rules
filter_rules(array $rules); // Get or set the filtering rules
run(array $data); // Runs the filter and validation routines
xss_clean(array $data); // Strips and encodes unwanted characters
sanitize(array $input, $fields = NULL); // Sanitizes data and converts strings to UTF-8 (if available)
validate(array $input, array $ruleset); // Validates input data according to the provided ruleset (see example)
filter(array $input, array $filterset); // Filters input data according to the provided filterset (see example)
get_readable_errors($convert_to_string = false, $field_class="field", $error_class="error-message"); // Returns human readable error text in an array or string
get_readable_error(array $e, $field_class); // Returns human readable text for a specific error
```

#  Complete Working Example

The following example is part of a registration form, the flow should be pretty standard

```php
# Note that filters and validators are separate rule sets and method calls. There is a good reason for this. 

require "gump.class.php";

$gump = new GUMP(); 

$_POST = $gump->sanitize($_POST); // You don't have to sanitize, but it's safest to do so.

$gump->validation_rules(array(
	'username'    => 'required|alpha_numeric|max_len,100|min_len,6',
	'password'    => 'required|max_len,100|min_len,6',
	'email'       => 'required|valid_email',
	'gender'      => 'required|exact_len,1|contains,m f',
	'credit_card' => 'required|trim|valid_cc'
));

$gump->filter_rules(array(
	'username' 	  => 'trim|sanitize_string|mysql_escape',
	'password'	  => 'trim|base64',
	'email'    	  => 'trim|sanitize_email',
	'gender'   	  => 'trim',
	'bio'		  => 'noise_words'
));

$validated_data = $gump->run($_POST);

if($validated_data === false) {
	echo $gump->get_readable_errors(true);
} else {
	print_r($validated_data); // validation successful
}
```

Return Values
-------------
`run()` returns one of two types:

*ARRAY* containing the successfully validated and filtered data when the validation is successful
*FALSE* when the validation has failed

`validate()` returns one of two types:

*AN ARRAY* containing key names and validator names when data does not pass the validation.

You can use this array along with your language helpers to determine what error message to show.

*A BOOLEAN* value of TRUE if the validation was successful.

`filter()` returns the exact array structure that was parsed as the `$input` parameter, the only difference would be the filtered data.


Available Validators
--------------------
* required `Ensures the specified key value exists and is not empty`
* valid_email `Checks for a valid email address`
* max_len,n `Checks key value length, makes sure it's not longer than the specified length. n = length parameter.`
* min_len,n `Checks key value length, makes sure it's not shorter than the specified length. n = length parameter.`
* exact_len,n `Ensures that the key value length precisely matches the specified length. n = length parameter.`
* alpha `Ensure only alpha characters are present in the key value (a-z, A-Z)`
* alpha_numeric `Ensure only alpha-numeric characters are present in the key value (a-z, A-Z, 0-9)`
* alpha_dash `Ensure only alpha-numeric characters + dashes and underscores are present in the key value (a-z, A-Z, 0-9, _-)`
* numeric `Ensure only numeric key values`
* integer `Ensure only integer key values`
* boolean `Checks for PHP accepted boolean values, returns TRUE for "1", "true", "on" and "yes"`
* float `Checks for float values`
* valid_url `Check for valid URL or subdomain`
* url_exists `Check to see if the url exists and is accessible`
* valid_ip `Check for valid IP address`
* valid_cc `Check for a valid credit card number (Uses the MOD10 Checksum Algorithm)`
* valid_name `Check for a valid format human name`
* contains,n `Verify that a value is contained within the pre-defined value set`

Available Filters
-----------------
Filters can be any PHP function that returns a string. You don't need to create your own if a PHP function exists that does what you want the filter to do.

* sanitize_string `Remove script tags and encode HTML entities, similar to GUMP::xss_clean();`
* urlencode `Encode url entities`
* htmlencode `Encode HTML entities`
* sanitize_email `Remove illegal characters from email addresses`
* sanitize_numbers `Remove any non-numeric characters`
* trim `Remove spaces from the beginning or end of strings`
* base64_encode `Base64 encode the input`
* base64_decode `Base64 decode the input`
* sha1 `Encrypt the input with the secure sha1 algorithm`
* md5 `MD5 encode the input`
* noise_words `Remove noise words from string`
* json_encode `Create a json representation of the input` 
* json_decode `Decode a json string` 
* rmpunctuation `Remove all known puncutation characters from a string`
* basic_tags `Remove all layout orientated HTML tags from text. Leaving only basic tags`

#  Creating your own validators and filters

Simply create your own class that extends the GUMP class.

```php
	
require("gump.class.php");

class MyClass extends GUMP
{
	public function filter_myfilter($value)
	{
		...
	}
	
	public function validate_myvalidator($field, $input, $param = NULL)
	{
		...
	}
	
} // EOC

$validator = new MyClass();

$validated = $validator->validate($_POST, $rules);

```

Remember to create a public methods with the correct parameter types and counts.

For filter methods, prepend the method name with "filter_".
For validator methods, prepend the method name with "validate_".

Running the examples:
------------------

1. Open up your terminal
2. cd [GUMP DIRECTORY/examples]
3. php [file].php

The output will depend on the input data.

# TODO

* Add composer compatibility
* A currency validator
* An address validator
* A country validator
* Location co-ordinates validator
* HTML validator
* Language validation ... determine if a piece of text is a specified language
* Validate a spam domain or IP.
* Validate a spam email address
* Validate spam text with askimet or something similar
* Improve documentation
* More examples
* W3C validation filter?
* A filter that integrates with an HTML tidy service?: http://infohound.net/tidy/
* Add a twitter & facebook profile url validator: http://stackoverflow.com/questions/2845243/check-if-twitter-username-exists
* Add more logical examples - log in form, profile update form, blog post form, etc etc.
