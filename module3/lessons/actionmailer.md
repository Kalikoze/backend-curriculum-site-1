---
layout: page
title: Sending Email Tutorial
length: 30
tags: rails, email, smtp, actionmailer
---

## Sending Email

We'll explore sending email in Rails by building a project that requires this functionality. By the end you should understand:

* How to use ActionMailer
* Send email locally using Mailcatcher
* How to setup a third party email service

### Friendly Advice

We are going to build an application that allows us to send our friend some friendly advice via email.

What we'd like our app to do:

1. Allow us to sign up / log in
2. Allow us to enter a friend's email address
3. Send that friend some advice.  

## Researching with the Docs  
* [What is SMTP?](http://whatismyipaddress.com/smtp)
* [Action Mailer](http://guides.rubyonrails.org/action_mailer_basics.html)  

## Sending Email locally with the mailcatcher

### Getting Started

For this tutorial, we are going to setup our emails to "send" through mailcatcher locally. And in production, the emails will actually send through Sendgrid. If you want to setup an account with Sendgrid in order to send emails both locally and in production, you can set up an account with them. Keep in mind that Sendgrid will provision your account at first so it might take some time to get the account set up. You should sign up for the free account.

First things first, go ahead and clone down [this repo](https://github.com/turingschool-examples/friendly-advice)

```shell
$ git clone https://github.com/turingschool-examples/friendly-advice.git friendly-advice
$ cd friendly-advice
$ bundle
$ rake db:{create,migrate}
```
### Rails Setup  
Run your server to see what we've already got in our repo
```
rails s
```  
Inspect the Form/Input field, where does this route to?

Make sure the route is in the routes file:

```rb
post '/advice' => 'advice#create'
```

Next we'll open our advice controller. The form asks for a post route so we'll need to update the create action where we will call our mailer.

```rb

# app/controllers/advice_controller.rb

class AdviceController < ApplicationController

  def show
   redirect_to root_path unless logged_in?
  end

  def create
    @advice = AdviceGenerator.new
    # `recipient` is the email address that should be receiving the message
    recipient = params[:friends_email]
    # `email_info` is the information that we want to include in the email message.
    email_info = { user: current_user,
                   friend: params[:friends_name],
                   message: @advice.message
                 }
    FriendNotifierMailer.inform(email_info, recipient).deliver_now
    flash[:notice] = "Thank you for sending some friendly advice."
    redirect_to advice_url
  end
end
```

### Creating the Mailer

Our next step will be to create the FriendNotifer mailer to send our friends advice.

```shell
rails g mailer FriendNotifier
```

This creates our `friend_notifer_mailer` inside `app/mailers` and a `friend_notifer_mailer` folder in `app/views`. Let's first open the `friend_notifer_mailer` in mailers and add our inform method.

```rb
# mailers/friend_notifier_mailer.rb

class FriendNotifierMailer < ApplicationMailer
  def inform(info, recipient)
    @user = info[:user]
    @message = info[:message]
    @friend = info[:friend]
    mail(to: recipient, subject: "#{@user.name} is sending you some advice")
  end
end
```

Next we'll make the views that will have the body of the email that is sent. Similar to controllers, any instance variables in your mailer method will be available in your mailer view.

When you generated your mailer, two layouts were added to `app/views/layouts` - `mailer.html.erb` and `mailer.text.erb`.  Take a look at these files, what are they doing? Why do we get both an HTML and text layout?

In `app/views/friend_notifier_mailer` create two files, `inform.html.erb` and `inform.text.erb`

Depending on the person's email client you're sending the email to, it will render either the plain text or the html view. We don't have control over that, so we want to accomodate both and make them have the same content.

```
# inform.html.erb and inform.text.erb

Hello <%=@friend%>!

<%= @user.name %> has sent you some advice: <%= @message %>
```

Let's also change the default address the email gets sent from:

```rb
# app/mailers/application_mailer.rb

class ApplicationMailer < ActionMailer::Base
  default from: "friendly@advice.io"
  layout 'mailer'
end
```

### Configuring Mailcatcher

Take a moment to see what you can figure out from the [MailCatcher](https://mailcatcher.me/) docs.

Mailcatcher allows you to test sending email. It is a simple SMTP server that intercepts (catches) outgoing emails, and it gives you a web interface that allows you to inspect them. Mailcatcher does not allow the emails to actually go out, and so you are able to test that emails send, without having to worry about that email being flooded with emails.

Let's get it set up.

You want to install the Mailcatcher gem, **but you do not want to add it to your gemfile** in order to prevent conflicts with your applications gems.

```sh
$ gem install mailcatcher
```

Now let's setup our development config to send the emails to the mailcatcher port. Add the following code to your `config/environments/development.rb` file.

```rb
# config/environments/development.rb

config.action_mailer.delivery_method = :smtp
config.action_mailer.smtp_settings = { :address => "localhost", :port => 1025 }
```

The mail is sent through port 1025, but in order to view the mailcatcher interface we want to visit port 1080. To start up mailcatcher:

```sh
$ mailcatcher
```

Now open up http://localhost:1080 and you should see the interface for where emails are sent. Now let's test to see emails come through.

If you open up the app in a browser locally, and run through the steps to send an email, you should see the email come through on mailcatcher.

### Materials

* [ Repo ](https://github.com/turingschool-examples/a_bit_of_advice)
* [ Slides ](https://www.dropbox.com/s/ev7tya328sv9jyh/Turing%20-%20Sending%20Email.key?dl=0)
* [Production Email Checklist](../lessons/production_email_checklist)
* [Testing Emails](https://guides.rubyonrails.org/testing.html#testing-your-mailers)
