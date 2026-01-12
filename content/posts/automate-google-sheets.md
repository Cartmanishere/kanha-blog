---
author: 'Pranav Gajjewar'
authorTwitter: ''
cover: '/images/automate-google-sheets/automate-emails-cover.png'
date: '2021-07-11'
description: 'Automate your workflows as a small business owner using Google Sheets & Appscript'
hideComments: false
keywords:
  - google-sheets
  - automation
  - emails
  - appscript
  - small business
  - billing
readingTime: false
showFullContent: false
tags:
  - blog
  - programming

title: Automate emails from Google Sheets
---

Google workspace has fantastic set of tools. Many people from small businesses to big companies use these tools to organize their data. But not many take advantage of its full extent of powers.

Today, I want to take a look at one instance where it can be useful. As per the title, we are going to look at how you can automate sending emails from Google Sheets. This might be a narrow but very real use case for people using Google Sheets.

Ultimately, the takeaway should be the amount of flexiblity you get from the tools we will use in this post and with a little bit of coding you can automate anything you want.

## Setup

Let’s explore a use case with a story (loosely based on a real requirement from a business I know).

I am running a small business where I take orders from people and put all the info in a Google sheet to track these orders.

You can imagine something like this:

{{< figure src="/images/automate-google-sheets/order-sheet-example.png" alt="Order sheet" position="center" caption="Order Sheet" captionPosition="center" >}}

I get orders from various forms and channels. And these orders are entered in this sheet only after I have vetted them.

So an entry in this sheet implies that the order has been accepted and is being worked upon.

But how does my customer know that the order is accepted and that the work has started? It would be a good idea to send them an email and let them know that their order has been accepted and a tentative delivery date would be awesome!

So I send them an email like this:

![](/images/automate-google-sheets/email-template-example.png)

Note: You would probably have a better looking email but this isn’t a design or marketing blog post 😛

For every confirmed order, I copy this email template and fill the required information in it from the sheet and then send the email to the customer. That’s a lot of work!

Also —

-   There is a high likelihood of making a mistake while doing this manually.
-   Wastes time for my employees when they can be doing better things than sending confirmation emails!

If any of you reading this blogpost make your employees send such emails manually —

![](/images/automate-google-sheets/facepalm-gif.gif)

Ok, so let’s see how we can automate this.

## Appscript 🚀

We will use [Google Appscript](https://www.google.com/script/start/) inside our Google Sheets to automatically send the emails.

Let’s build this system bottom-up so we can clearly understand what’s happening at each level.

-   Sending text emails from Appscript.
-   Rendering HTML templates for sending the emails.
-   Integrate this with Google Sheet.
-   Trigger the email using various ways.

This is the structure for the rest of this post.

### Sending Emails

Before we move to triggering emails and using HTML templates, let’s first understand _how_ exactly can we send emails from Appscript.

To get started, you have to open the script editor. In your Google Sheet menu, go to **Tools > Script Editor.**

This should open a new Appscript editor window. In the left hand side, you can see that we have opened a file `code.gs`. We will write all our code in this file.

Google Appscript uses Javascript syntax. It uses [Rhino](https://github.com/mozilla/rhino) engine to run the code. So if you know javascript, then you can write Appscript as well.

In this editor, we can write functions that we can run. Let’s write a function as follows:

![](/images/automate-google-sheets/send-email-function.png)

Replace the email address with a real one and click `Run`. When it finishes running successfully, you can check the inbox to see whether you received the email or not.

Note: First time, it might ask you to authorize since it needs permissions to send emails on your behalf.

### Rendering HTML

We don’t want to send text emails though. For any business, it is important to show a good UI to the customer.

In this case, we have an HTML template using which we want to send email. And we want to fill the data in the template from our script i.e. the customer information is not hardcoded in the template.

Let’s see how that can be done here.

This is my HTML template from the above example but I am using placeholders instead of actual customer information.

[Full template](https://gist.github.com/Cartmanishere/43f47fd9bb7de096ea6b2900980761a5) (you can create your own as well, just use the following variables in the HTML)

<h1 style="...">
  Hi <?= context.customerName ?>!
</h1><p style="..."><?= context.message ?></p><tbody>
  <tr>
    <td class="tg-0lax"><?= context.customerName ?></td>
    <td class="tg-0lax"><?= context.status ?></td>
    <td class="tg-nh17"><?= context.deliveryDate ?></td>
  </tr>
</tbody>

Appscript allows much more logic in the HTML templates. But in this case we will only use it for substituting variable values. For more details, you can refer [HTML Service: Templated HTML](https://developers.google.com/apps-script/guides/html/templates#index.html_5)

But how can we use this template in our Appscript?

1.  From the left sidebar menu, click on `+` to add a new `File`
2.  Select an HTML file and name it `emailTemplate`
3.  Paste all the HTML template code inside that file.

Now, we can write a function as follows to render this HTML template with data from Appscript.

![](/images/automate-google-sheets/render-html-template-function.png)

The above function gets the email template, then renders the template with the data we provide.

In above case, `<?= context.message ?>` will be replaced by the `message` value we assign from the Appscript.

Now, let’s put the earlier function and this one together to see how we can send emails with HTML template rendered content.

Running the `sendEmail` function again will send you an email with rendered HTML.

So far, we have been harcoding the customer information in our code. Let’s see next how these value can be read from our original spreadsheet.

### Integrate with Sheet

I intentionally added one more function `sendCustomerNotification` in the above snippet and defined the order info in an array. Because when we read from the actual sheet, we will also get the information in the form of an array.

So let’s see how that works.

There are different ways you can read the info from the sheet. But this is how we will read it here.

In our original Google Sheet, we can select a range of rows like:

![](/images/automate-google-sheets/sheet-range-selection.png)

And we want to send emails for the selected rows from our Appscript.

We can write a function for this as follows:

![](/images/automate-google-sheets/read-sheet-data-function.png)

And we can see in the execution log after running above function —

Press enter or click to view image in full size

![](/images/automate-google-sheets/execution-log-output.png)

So that’s how the selected range of rows from our sheet can be accessed in our Appscript.

To briefly explain, what’s happening:

-   We get the currently active spreadsheet using `SpreadsheetApp.getActiveSpreadSheet()` (Note: You can get spreadsheet using the id as well)
-   We get the currently active Sheet (inside active Spreadsheet) using `spreadsheet.getActiveSheet()` (Note: you can get the sheet using name as well)
-   We get the selected rows by using `sheet.getActiveRange()`

When we loop over `selection.getValues()` we get all the info in the row in the form of an array as seen above.

We can directly replace `console.log(rows[i])` with `sendEmail(rows[i])` in the above snippet and we are done.

When we run the function `sendCustomerNotification`, it will read the selection of rows from the Sheet and send email to each customer.

### Triggering the function

Now, to send the emails from our sheet, we have to select the rows for which we want to send email and then open script editor and run the `sendCustomerNotification` function.

This is a roundabout way of doing things. Can we not simply send the emails from the Google Sheet?

Yes! There are multiple ways this can be done. I’ll demonstrate two ways which I think will be useful.

**Using a button**

We can simply add a button in our sheet which upon clicking will send emails to all the selected customers.

Go to **Insert > Drawing** from the Sheet Menu. And upload any button image that you like. E.g.

Press enter or click to view image in full size

![](/images/automate-google-sheets/button-image-example.png)

Place that image somewhere on the sheet. And then click the 3-dot menu you see on the image.

There you can select the option `Assign script` and then enter the function name `sendCustomerNotification`. That’s it!

Now, whenever you want to send emails, you can select the rows and then just click on the above button and it will run the `sendCustomerNotification` function and send all the emails.

**Using a macro**

If you don’t want a button on your sheet, then that’s fine. Using a macro is an even easier option to send the emails.

Here’s how you can setup a macro for your sheet.

Go to **Tools >** **Macros > Import,** then select the `sendCustomerNotification` function from the function list.

After adding the macro, go to **Tools > Macros > Manage macros** and you can set up a keybinding for the imported macro as follows:

![](/images/automate-google-sheets/macro-keybinding-setup.png)

Update this keybinding and you’re done.

Now on the sheet, you just have to select the range of rows for which you want to send notification and press `CMD + OPTION + SHIFT + 9` and it will execute the `sendCustomerNotification` function.

Pretty handy right!

Here’s a short demo:

![](/images/automate-google-sheets/demo-gif.gif)

I set the background to green after sending all the emails in the `sendCustomerNotification` function just so I know everything worked successfully.

Understanding a little bit of coding can take you very far. Just based on the things we used in this post, you can mix and match to automate other things you want.

You can create a set of workflows for yourself and for your team. You can automate almost anything you want with Appscript. The efforts required may vary.

Hope this post was helpful 😄
