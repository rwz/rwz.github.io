---
layout: post
title: Datetime select
---

Consider this form

{% highlight haml %}
= form_for User.new do |form|
  = form.label :birthday
  = form.date_select :birthday
{% endhighlight %}

This typically generates HTML similar to this:

{% highlight html %}
<form accept-charset="UTF-8" action="/users" method="post">
  <label for="user_birthdate">Birthdate</label>
  <select id="user_birthdate_3i" name="user[birthday(3i)]"><option>Day</option>
  <select id="user_birthdate_2i" name="user[birthday(2i)]"><option>Month</option>
  <select id="user_birthdate_1i" name="user[birthday(1i)]"><option>Year</option>
</form>
{% endhighlight %}

There are two problems with this code:
  - The label does not actually point to any of the selects. Select ids are
    different from labels for attribute.
  - There are 3 separate selects for a single database fields that could go in
    any arbitrary order.

Both problems make these types of fields super inconvenient to access from
capybara/cucumber tests. Your trivial `select "1986/08/25", from: "Birthday"`
is not gonna work.

So, I came up with custom step for these kind of things:


{% highlight ruby %}
When /^I fill in "(.*?)" date field with "(.*?)"$/ do |field_name, date_components|
  label = find("label", text: field_name)
  select_base_id = label[:for]
  date_components.split(",").each_with_index do |value, index|
    select value.strip, from: "#{select_base_id}_#{index+1}i"
  end
end
{% endhighlight %}


So you can use it like this:

{% highlight gherkin %}
When I fill in "Birthdate" fate field with "25, Aug, 1986"
{% endhighlight %}
