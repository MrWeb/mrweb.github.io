---
layout: post
title:  "PHP vCard iOS and mobile download"
date:   2020-05-27 12:24:48 +0200
categories: php
---

I recently struggled in trying to implement a vCard download on iOS from a PHP web application, and this is the reason for this post.

I'm using the package <a href="https://github.com/jeroendesloovere/vcard" target="_blank">https://github.com/jeroendesloovere/vcard</a> with the function `download()` this caused a bug on the iPhone where the VCF gets turned into a VCF.html and is not readable.

This is a method I was using that caused the issue:

```php
$contact = ContactModel::where('id', $contact_id)->firstOrFail();

$lastname  = $contact->lname;
$firstname = $contact->fname;
$prefix    = $contact->title;

$vcard = new VCard();
$vcard->addName($lastname, $firstname, null, $prefix, null);
$vcard->addPhoneNumber($contact->phone_1, 'PREF;WORK');
$vcard->addPhoneNumber($contact->phone_2, 'WORK');
$vcard->addEmail(strtolower($contact->email_1));
$vcard->addEmail(strtolower($contact->email_2));
$vcard->addAddress(null, null, $contact->address ?? '', $contact->zone->name ?? '', null, $contact->zone->ZIP ?? '', $contact->zone->country ?? '');

$vcard->download();
```

I found out the problem is because of wrong headers sent by the `download()` function in the `vcard` package.

To solve this issue you can manually set the headers (I'm using Laravel in this project but this applies to any plain PHP project as well) and instead of using the `download()` function we just return the output:

```php
#same code as above but instead of $vcard->download(); we now do

$response = new Response();
$response->setContent($vcard->getOutput());
$response->setStatusCode(200);
$response->headers->set('Content-Type', 'text/x-vcard');
$response->headers->set('Content-Disposition', 'attachment; filename="' . $contact_id . '.vcf"');
$response->headers->set('Content-Length', mb_strlen($vcard->getOutput(), 'utf-8'));

return $response;
```