---
layout: post
title: "Running cucumber features in different locale"
date: 2014-01-04 10:59:30 +0530
comments: true
categories: 
---
We use Selenium + Capybara + Firefox to run all our cucumber features. Recently, we decided to extend the tests to run in different locales and this post is a summary of the problems and the plausible solutions.

We had the following objectives in mind:

* Since, we run the tests in firefox, the test process should be capable of creating an appropriate profile with required locale settings
* The tests should be generic enough so that they don't have to be written and maintained separately for every locale

## Setup

The first step involves creating a new browser profile and switching the locale (italian in this case) and passing it on to the Capybara driver.

``` ruby env.rb
Capybara.default_driver = :selenium

it_profile = Selenium::WebDriver::Firefox::Profile.new
it_profile['intl.accept_languages'] = 'it' # Italian Locale

Capybara.register_driver :selenium do |app|
  Capybara::Selenium::Driver.new(app, :browser => :firefox,
                                      :profile => it_profile)
end

Capybara.current_session.driver.browser.capabilities.firefox_profile = it_profile
```
The advantage of creating a profile on-the-fly is that the test does not demand every machine to have a pre-configured firefox profile that has the necessary settings.

<!-- more -->

## Using translation keys in step definitions

Inside step definition, instead of hard-coding text, use the translation keys directly. This is a good practice even when writing tests for a single locale as your tests are not affected by textual changes.

``` ruby alert_steps.rb
When /^I click on the stock alert button$/ do |arg|
	find(button, visible: true, text: t('stock_tab.alert_caption')).click
end
```

When it's a step with text as parameter, use the key directly in the feature. Yes, this is not so readable as using the actual text but using sensible translation keys will take it one step closer to using actual text.
``` cucumber admin.feature 
Given I am logged in as administrator
And I click on t(settings)
```

The key passed as parameter can be used with I18n directly.
``` ruby admin_steps.rb
And /I click on t\(:?([^\)]*)\)/ do |key|
  find(button, visible: true, text: t(key)).click
end
```