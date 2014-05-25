---
layout: post
title: "Handling Ajax Redirects in Rails"
date: 2014-05-25 21:49:19 +0530
comments: true
categories: rails, javascript
---

Apparently there is no clean way to do ajax redirects in Rails. The recommended way to do that is to send a regular `200 OK` status code along with some special parameter signaling the javascript side that it is a `REDIRECT`.

Now, what follows is a relatively cleaner way to handle redirection in ajax calls. 

On the rails side, we create a helper that returns a response that has no content but just one header `x_ajax_redirect_url` and status code 302.

``` ruby application_helper.rb 
    def ajax_redirect_to(url)
        head 302, x_ajax_redirect_url: url
    end
```

A global ajax handler on the javascript side will intercept all ajax calls across the application and redirect on seeing the appropriate header construct.

``` javascript application.js 
	$.ajaxSetup({
        statusCode: {
            302: function (response) {
                var redirect_url = response.getResponseHeader('X-Ajax-Redirect-Url');
                if (redirect_url != undefined) {
                    window.location.pathname = redirect_url;
                }
            }
        }
    });
```

Now, let's see this put to use in the most common scenario of validating the user session. If an invalid session is encountered and the request is xhr (meaning ajax), `ajax_redirect_to` can be used to redirect the user to login page else the conventional `redirect_to` helper should work.

``` ruby application_controller.rb
class ApplicationController < ActionController::Base
	
	before_filter :check_user_session, except: [:logout]

	def check_user_session
		unless valid_user?
			request.xhr? ? ajax_redirect_to(login_path) : redirect_to login_path
		end
	end

end
```

> Interestingly, `redirect_to url, status: 302` is not the same as using `head 302, x_ajax_redirect_url: url` because the latter does not set the location header and that does the trick! 