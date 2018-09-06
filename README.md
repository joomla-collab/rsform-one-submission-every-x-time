# RSForm
## One submission every X time

With this code, we can check if the user has submitted a form previously in the set amount of time.  
This code checks a public form. If you want to check against an existing user, it would be actually easier.

Start by creating a checkbox, and hiding all visible elements with CSS, but not touching the validation message.  
RSForm will display the validation message if the conditions match in the code below.
Make the checkbox a single value, and add [c] to the end of it's value to make it checked by default, and make it mandatory.

Here is the basic code:

```
// Date 24 hours ago
// Do a google search to find out how to set PHP's strtotime. 
$lastDay = date('Y-m-d H:i:s', strtotime('-24 hours'));

// Define your field to be compared, and use the field name from your form.
$fieldPhone = "Phone";

// Find the form ID for your form
$formID = "1";

// Check if the submission was successful
$confirmedForm = "1";

// Get the value from the current form
$phone = $_POST['form']['Phone'];

// Initialize the database query
$db = JFactory::getDbo();
$query = $db->getQuery(true);
// Get the submission ID for matching
$query->select('SubmissionId');
// Select the submission_values table
$query->from($db->quoteName('#__rsform_submission_values'));
// Set the name of the field to be matched. Attention: this is a variable set above
$query->where($db->quoteName('FieldName')." = ".$db->quote($fieldPhone));
// Set the value of the field to be matched
$query->where($db->quoteName('FieldValue')." = ".$db->quote($phone));
// Set the form ID to be matched
$query->where($db->quoteName('FormId')." = ".$db->quote($formID));
// Limit the check to just the latest one
$query->setLimit('1');
// Order by submission ID, so you can get the latest one
$query->order('SubmissionId DESC');
$db->setQuery($query);
// Get the resulting submission ID, if any
$subPhone = $db->loadResult();

// Check if the previous query returned something
if($subPhone) {
  // Do a whole new database query, this time against the submissions table
	$db = JFactory::getDbo();
	$query = $db->getQuery(true);
  // Now we need the date it was submitted
	$query->select('DateSubmitted');
	$query->from($db->quoteName('#__rsform_submissions'));
  // Select the submission ID from the previous query
	$query->where($db->quoteName('SubmissionId')." = ".$db->quote($subPhone));
	$query->where($db->quoteName('FormId')." = ".$db->quote($formID));
	$query->where($db->quoteName('confirmed')." = ".$db->quote($confirmedForm));
	$query->setLimit('1');
	$query->order('DateSubmitted DESC');
	$db->setQuery($query);
  // Get the resulting submission ID
	$subDateEmail = $db->loadResult();
  // Check if this latest query returned something, and check if it is higher than the time limit set above
	if($subDateEmail && $subDateEmail >= $lastDay) {
    // If it is a match, then set it as true for further checks
		$matchPhone = true;
	}
}  
  
// You can repeat the previous pair of queries to check against other fields  

// This if statement can be if something OR other thing...
if($matchPhone == true) {
  // If there is a match, we invalidate a field in the form, in order to display an error message and prevent the form from being sent.
	$invalid[] = RSFormProHelper::getComponentId("checktimer");
}
```

And that's it.
