---
layout: post
title: Turbolinks with Devise
date: 2017-01-29 03:20
categories:
---

If you've ever tried to have your Devise forms submit via AJAX, then you may have found that the resources to accomplish this are kind of spread out all over the internet. 

Consider this my humble effort to bring all those bits and pieces of information together in a way that I wish was available when I took on this task. Without further ado, here are the steps I took to use Devise with Turbolinks. 

_Note: This assumes you already have a fairly default configuration of Devise._

First, we have to configure Devise to allow us to use [jquery-ujs](https://robots.thoughtbot.com/a-tour-of-rails-jquery-ujs) and [SJR](https://signalvnoise.com/posts/3697-server-generated-javascript-responses) responses.

{% highlight ruby  %}
# config/initializers/devise.rb

# If 401 status code should be returned for AJAX requests. True by default.
config.http_authenticatable_on_xhr = false

# ==> Navigation configuration
# Lists the formats that should be treated as navigational. Formats like
# :html, should redirect to the sign in page when the user does not have
# access, but formats like :xml or :json, should return 401.
#
# If you have any extra navigational formats, like :iphone or :mobile, you
# should add them to the navigational formats lists.
#
# The "*/*" below is required to match Internet Explorer requests.
config.navigational_formats = ['*/*', :html, :js]
{% endhighlight %}

{% highlight ruby  %}
# app/controllers/application_controller.rb

class ApplicationController < ActionController::Base
  respond_to :html, :js, if: :devise_controller?
end
{% endhighlight %}

On to the views, if you have not already copied the devise views into your application, let's go ahead and do that.

{% highlight shell  %}
rails generate devise:views
{% endhighlight %}

Now we'll use jquery-ujs to easily make all the devise view forms submit via AJAX. Here's an example.

{% highlight erb  %}
<!-- app/views/devise/registrations/new.html.erb -->

<%= form_for(resource, as: resource_name, url: registration_path(resource_name), remote: true) do |f| %>
{% endhighlight %}

Voila! Moving forward, we come to the SJR responses. Since our forms are now submitted via AJAX, we need to handle those responses properly. Here's an example.

{% highlight javascript %}
// app/views/devise/registrations/create.js.erb

<% if user_signed_in?  %>
  Turbolinks.clearCache()
  Turbolinks.visit("<%= response.location %>", { "action": "replace" })
<% else %>
  $('[data-behavior="devise-error-messages"]').html('<%= j devise_error_messages! %>');
  $('[data-behavior="flashes"]').html('<%= j render "layouts/flashes", flash: flash %>');
<% end %>
{% endhighlight %}

For most of the responses you'll only have to deal with create/update, but when you try to write the response for your login, you'll [find an odd issue](http://stackoverflow.com/questions/22689877/devise-ajax-login-sessionscreate-only-render-create-js-erb-when-it-succeed/31713225#31713225). It's a small quirk that just means we'll have to separate our logic from the previous example.

{% highlight javascript %}
// app/views/devise/sessions/new.js.erb

$('[data-behavior="devise-error-messages"]').html('<%= j devise_error_messages! %>');
$('[data-behavior="flashes"]').html('<%= j render "layouts/flashes", flash: flash %>');
{% endhighlight %}

{% highlight javascript %}
// app/views/devise/registrations/create.js.erb

Turbolinks.clearCache()
Turbolinks.visit("<%= response.location %>", { "action": "replace" })
{% endhighlight %}

Finally, we'll set up our flashes such that we get proper error messages from the responses. Let's add that anywhere it makes sense in your layout.

{% highlight erb %}
<!-- app/views/layouts/application.html.erb -->

<div data-behavior="devise-error-messages">
  <%= devise_error_messages! %>
</div>
<div data-behavior="flashes">
  <%= render partial: "layouts/flashes", local: { flash: flash } %>
</div>
<%= yield %>
{% endhighlight %}

And just like that we have Devise working in perfect harmony with Turbolinks!