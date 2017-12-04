# API

An Application Programming Interface (API) is a means of providing programming hooks into a larger system. For example ````document.getElementById('el').style```` is a programming hook that allows a JavaScript programmer to work with the browser. Some APIs are open to the public and others are closed and accessible only by those who have been granted special access. Some APIs are accessible over Internet protocols.

## JSON

Before we dive into the Mailgun API we should take a moment to talk about JavaScript Object Notation (JSON). This is the notation used by JavaScript for creating objects it is a string of key pairs (similar to an array) and in recent years has become the defacto standard for interacting with API. Since we are using PHP to interact with JSON we will use [````json_decode()````](http://php.net/manual/en/function.json-decode.php) and [````json_encode()````](http://php.net/manual/en/function.json-encode.php) to convert a JSON string into an array and vice versa.


Curly braces _{}_ encapsulates a JSON string (creates an object in JS)

Colons _:_ separate key to value pairs, fields are comma separated (aka CSV or comma separated values).
````
{
  '_id':1234,
  'firstname':'Sally',
  'lastname':'Smith'
}
````

A company with multiple addresses (in this case an array of addresses)
````
{
  '_id'':1,
  'company':'MicroTrain Technologies',
  'addresses':[
    {
      'street':'200 W Adams Suite 410',
      'city':'Chicago',
      'state':'IL',
      'zip':60606
    },
    {
      'street':'720 E Butterfield Rd Suite 100',
      'city':'Lombard',
      'state':'IL',
      'zip':60148
    },
  ]
}
````

One JSON string with multiple company objects
````
{
  {
    '_id'':1,
    'company':'42 North Group, Inc,
    'addresses':[
      {
        'street':'720 E Butterfield Rd Suite 100',
        'city':'Lombard',
        'state':'IL',
        'zip':60148
      },
    ]
  },
  {
    '_id'':2,
    'company':'TheProfessional.Me',
    'addresses':[
      {
        'street':'200 W Adams Suite 410',
        'city':'Chicago',
        'state':'IL',
        'zip':60606
      },
      {
        'street':'720 E Butterfield Rd Suite 100',
        'city':'Lombard',
        'state':'IL',
        'zip':60148
      },
    ]
  }
}
````

In any case PHP will convert this to an array and you would parse out the array to find your desired value. So the first example would appear as.
````
array(
  '_id' => 1234,
  'firstname' => 'Sally',
  'lastname' => 'Smith'
)
````

Let's this was returned via a fictitious _User->get(1234)_ method (in which User is a class and get() is a member (property/method) of that class) you might access _firstname_ as follows.
````
$users = new User();
$user = $users->get(1234);

//Writes Sally to the screen
echo $user['firstname'];
````

The Mailgun SDK abstracts most of the JSON interaction away from us, so we are left working mostly with PHP objects. JSON will come up again so this seems a good time introduce it.

## Mailgun API

### Exercise 1 - Getting Started with Mailgun.

Go to [mailgun.com](https://www.mailgun.com/) and create a free account. You may choose not to add a credit card as we will be working with the sandbox. You will need to activate your account via email and SMS, this requires a phone number that receive texts. If you do not have one try creating a Google Voice account.

The landing page will provide you the details you need need to send a test email. Start by creating a shell script and pasting the curl example into the shell.

````
vim ~/mailgun.sh
````

Add the line

````
#!/bin/bash
````

Then
* Copy and paste the curl command into the file.
* Make the file executable.
* Execute the script.

````
chmod +x mailgun.sh
./mailgun.sh
````

If you see a [json](http://www.json.org/) response similar to this, your sand box account is working.
````
{
  "id": "<20171009154755.125901.xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.mailgun.org>",
  "message": "Queued. Thank you."
}
````

### Exercise 2 - Mailgun and PHP

The first thing you will want to do is [Mailgun's PHP](https://github.com/mailgun/mailgun-php) library to your project. You will do this via Composer.

Let's start by creating a feature branch. This will allow us to keep any changes separate from our production code until we are ready for it.

````
cd /var/www/about
git checkout -B feature/mailgun
````

If you get the message *Switched to a new branch 'feature/mailgun'* you are ready to proceed.
```
composer require mailgun/mailgun-php php-http/curl-client guzzlehttp/psr7
````

This may take a minute or so. The install was successful if you see a series of *Installing* messages ending with
````
Writing lock file
Generating autoload files
````

Now, let's see what was installed.
````
git status
````

You should see something like the following.
````
composer.json
composer.lock
vendor/
````

We do not want to commit the vendor directory to our repo. So we will create a *.gitignore* file. From you Atom sidebar create a file called *.gitignore* under the about project (Do not forget the preceding dot) and add the following line.
````
/vendor/
````

Now if you run ````git status```` you will see the following.
````
.gitignore
composer.json
composer.lock
````

Commit these files and push to the new feature branch.
````
git add .
git commit -m 'Added mailgun lib'
git push origin feature/mailgun
````

Create the file */var/www/YOUR-PROJECT-NAME/test.php* and copy and paste the PHP sample code from the Mailgun landing page. Below the pasted code add the line ````var_dump($reults);````.

````
<?php
# Include the Autoloader (see "Libraries" for install instructions)
require 'vendor/autoload.php';
use Mailgun\Mailgun;

# Instantiate the client.
$mgClient = new Mailgun('key-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx');
$domain = "sandboxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.mailgun.org";

# Make the call to the client.
$result = $mgClient->sendMessage("$domain",
          array('from'    => 'Mailgun Sandbox <postmaster@sandboxdxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.mailgun.org>',
                'to'      => 'YOUR-NAME <YOUR-EMAIL-ADDRESS>';
                'subject' => 'Hello YOUR-NAME',
                'text'    => 'Congratulations YOUR-NAME, you just sent an email with Mailgun! You are truly awesome! '));

var_dump($reults);
````

From a browser window, navigate to *http://localhost/YOUR-PROJECT-NAME/test.php* and you should get a json string similar to:
````
object(stdClass)#24 (2) { ["http_response_body"]=> object(stdClass)#19 (2) { ["id"]=> string(91) "<20171009164718.79178.xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.mailgun.org>" ["message"]=> string(18) "Queued. Thank you." } ["http_response_code"]=> int(200) }
````
Then check your email to see if it worked.


**Security Check Point**
_Never push a key to a public repository, use a key file the exists outside of the public repo_

## Exercise 2 - Pass Variables into the API Call

Create a key file
````
mkdir -p /var/www/YOUR-PROJECT-NAME/config
vim /var/www/YOUR-PROJECT-NAME/config/keys.php
````

Add the following line to your .gitignore file and commit your changes to the *feature/mailgun* branch.

````
/config/keys.php
````

Add the following (Where YOUR-KEY-HERE is the key provided by Mailgun):
````
<?php
define('MG_KEY', 'dYOUR-KEY-HERE');
define('MG_DOMAIN', 'YOUR-DOMAIN-HERE');
````
Add the following to the top of *http://localhost/YOUR-PROJECT-NAME/test.php*
````
require '/var/www/YOUR-PROJECT-NAME/config/keys.php';
````

Change
````
$mgClient = new Mailgun('key-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx');
$domain = 'sandboxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.mailgun.org';

$from = 'Mailgun Sandbox <postmaster@xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.mailgun.org>';
````
to
````
$mgClient = new Mailgun(MG_KEY);
$domain = MG_DOMAIN;

$from = "Mailgun Sandbox <postmaster@{$domain}>";
````

Instead of passing static values into the method pass variables.
* Replace the existing method call with the following.
* Remove the var_dump($results);

````
$from = "Mailgun Sandbox <postmaster@{$domain}>";
$to = 'YOUR-NAME <YOUR-EMAIL-ADDRESS>';
$subject = 'Hello YOUR-NAME';
$text = 'Congratulations YOUR-NAME, you just sent an email with Mailgun! You are truly awesome!';

$result = $mgClient->sendMessage(
  $domain,
  array('from'    => $from,
        'to'      => $to,
        'subject' => $subject,
        'text'    => $text
    )
);
````
From a browser window, navigate to *http://localhost/YOUR-PROJECT-NAME/test.php* and check your email.

## Additions Resources
* [JSON](http://www.json.org/)
* [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON)
### Udemy
* [JSON and APIs](https://microtrain.udemy.com/json-course/)