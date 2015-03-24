# ProcessWire SimpleContactForm

## Overview

Just a simple contact form. Optional support for Twig ([TemplateTwigReplace](http://modules.processwire.com/modules/template-twig-replace)) as template engine. Not more and not less.

Designed for use with [ProcessWire](http://processwire.com) version 2.5  
Current version: 0.1.0 stable

## Installation

This module will create templates. Please make sure that permissions of `site/templates` are set to 0777. After successful installation you can undo this.

1. Clone the module and place SimpleContactForm in your `site/modules/` directory. 

	```
	git clone https://github.com/justonestep/processwire-simplecontactform.git your/path/site/modules/SimpleContactForm
	```

2. Login to ProcessWire admin and click Modules.
3. Click "Check for new modules".
4. Click "install" next to the new SimpleContactForm module. 
5. Enter settings similar to the example below:

| setting                       | description                                                                                                                                                                                         |
|-------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| saveMessages                  | whether to save received messages                                                                                                                                                                   |
| useTwig                       | Check if you use Twig as template engine                                                                                                                                                            |
| fullName                      | Firstname Lastname                                                                                                                                                                                  |
| emailTo                       | xxx@xxx.xx                                                                                                                                                                                          |
| emailSubject                  | New Web Contact Form Submission                                                                                                                                                                     |
| successMessage                | Thank you, your submission has been sent                                                                                                                                                            |
| errorMessage                  | Please verify the data you have entered                                                                                                                                                             |
| errorMessage                  | noreply@xxx.xx                                                                                                                                                                                      |
| allFields                     | fullName,email,message                                                                                                                                                                              |
| requiredFields                | fullName,email,message                                                                                                                                                                              |
| emailField                    | email                                                                                                                                                                                               |
| antiSpamTimeMin               | It parses the time the user needs to fill out the form.  If the time is below a minimum time, the submission is treated as Spam.                                                                    |
| antiSpamTimeMax               | It parses the time the user needs to fill out the form.  If the time is over a maximum time, the submission is treated as Spam.                                                                     |
| antiSpamPerDay                | How often the form is allowed to be submitted by a single IP address in the last 24 hours.                                                                                                          |
| antiSpamExcludeIps            | Comma-Seperated list of IP addresses to be excluded from IP filtering.                                                                                                                              |
| antiSpamCountAdditionalInputs | Number of additional inputs. Spam bots often send more than the number of available fields. Default 5 (scf-date + scf-website + submitted + token + submit). AllFields will be added automatically. |


## Usage

1. Create a template for your contact form page (if you don't already have one).
2. Add the fields you want to use to this template as you are used to.
3. In the template add just one line to include the form:

	```php
	{{modules.get('SimpleContactForm').render()}} // twig
	$content = $modules->get('SimpleContactForm')->render(); //php
	```

4. If you want to send the form via ajax, include jquery.simplecontactform.js and call it:

	```javascript
	if ($('.js-simplecontactform').length) {
		$.simplecontactform($('.js-simplecontactform'));
	}
	```

	To get just the necessary part, modify yout template like this:

	```html
	{% if config.ajax %}
		{{modules.get('SimpleContactForm').render()}}
	{% else %}
		{# normal stuff ... #}
	{% endif %}
	```
	
**Note:** You have to include the module in your own template file first. After that, reload a frontend page using this template. After that, the additional templates are available.
	
### New Template "simple_contact_form"

* change type="input" due to your own needs
* hide following fields using css: scf-website, submitted
* make sure to maintain the names of the fields

tl;dr: These fields will be rendered automatically.
Therefore a new template called `simple_contact_form.[twig|php]` will be created in your `site/templates` directory.
All fields in this new template will be simple inputs temporarily.
Once created you can/should modify the template as well as the fields to your own needs, 
just make sure to maintain the names of the fields. 

### saveMessages enabled? 

* new page **scf-messages** (your-url.xx/scf-messages)
* new template **simple_contact_form**

tl;dr: If `saveMessages` is enabled, a new page `scf-messages` will be created.
Also a new repeater field containing all set up fields is added.
For each received message a new repeater element is stored.
The new template `simple_contact_form_messages.[twig|php]` which is created for this page checks 
whether the current user is logged into the backend or not.
If that is the case all received messages will be listed.
Otherwise the user will be redirected to the root page.
Failing that the user will be redirected to the root page.
You can modify that templates for your own needs.

## Logging

This module creates two log files.

* simplecontactform-log.txt
* simplecontactform-spam-log.txt

## Mark messages as spam

If the save messages setting is turned on you have the possibility to mark received messages as spam.
There are two options:

* mark as spam by mail address
* mark as spam by ip address

To mark a message as spam go to `Pages Tree > scfmessages > edit`. Each message has two checkboxes to mark either the email address as spam or the ip address.

In order to get the latest message to the beginning of the list, click the **⇧ Sort Inverse** at the right corner of the list.

## CSRF token validation

This module uses CSRF token validation, if you don't know what it's all about, have a look [at the ProcessWire Forum](https://processwire.com/talk/topic/3779-use-csrf-in-your-own-forms/).

## Upgrading from <= 0.0.9

If you upgrade an existing installation from 0.0.9 and below to the current version, there are some steps to be taken.

1. Upgrade the module source.
2. Visit the contact page in the frontend to receive all necessary database updates.
3. Edit page **scfmessages**, go to tab **Settings** and uncheck `Status Locked: Not editable` to be able to mark messages as spam.
4. Edit template file `simple_contact_form`. Due to implemented CSRF validation change:

	```php
	- <input type="hidden" name="TOKEN819808161X1427202408" value="D/iICidOcpcXHyd0lKdDs84qEtNnK..41" class="_post_token">
	+ <input type='hidden' name='<?= $input->tokenName; ?>' value='<?= $input->tokenValue; ?>' class='_post_token' /> // php
	+ <input type='hidden' name='{{input->tokenName}}' value='{{input->tokenValue}}' class='_post_token' /> // twig
	```

5. Edit template file `simple_contact_form_messages` and change the following to receive all messages except the ones marked as spam:
 
	```php
	- <?php foreach ($currentPage->repeater_scfmessages->sort('-scf-date') as $message) { ?> // php
	+ <?php foreach ($currentPage->repeater_scfmessages->find("scf_spamIp=,scf_spamMail=")->sort('-scf-date') as $message) { ?> // php
	- {% for message in currentPage.repeater_scfmessages.sort('-scf_date') %} // twig
	+ {% for message in currentPage.repeater_scfmessages.find('scf_spamIp=,scf_spamMail=').sort('-scf_date') %} // twig
	```

## Screenshots:

**Module Settings**

![Backend](https://github.com/justonestep/processwire-simplecontactform/blob/master/screens/settings.png)

**Example scf-messages**

![Backend](https://github.com/justonestep/processwire-simplecontactform/blob/master/screens/received-messages.png)
