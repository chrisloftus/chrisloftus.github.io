---
layout: post
title:  How To Force Remember Me Login In Your Laravel 5 Application
date:   2016-05-08 23:00:00
categories: laravel
---
Because no one likes being logged out, or having to check "Remember Me" when
they login.

So, how do we make Laravel always keep a user logged in?

First of all we can get rid of the "remember" input on the login page (if you
have one).

Secondly we need to take a look at how Laravel handles authentication. So we
take a look in our AuthController.php.

Here we can see two references to `AuthenticatesAndRegistersUsers`:

```php
<?php

use Illuminate\Foundation\Auth\AuthenticatesAndRegistersUsers;

class AuthController extends Controller {
    use AuthenticatesAndRegistersUsers, ThrottlesLogins;
    // ...
}
```

This must hold all of the authentication logic. So let's check out the trait.

We can do this by navigating to the file.

vendor/laravel/framework/src/Illuminate/Foundation/Auth/AuthenticatesAndRegistersUsers.php

```php
<?php

namespace Illuminate\Foundation\Auth;

trait AuthenticatesAndRegistersUsers
{
    use AuthenticatesUsers, RegistersUsers {
        AuthenticatesUsers::redirectPath insteadof RegistersUsers;
        AuthenticatesUsers::getGuard insteadof RegistersUsers;
    }
}
```

Here we can see that the `AuthenticatesAndRegistersUsers` trait *includes*
another trait: `AuthenticatesUsers`. Open this file (it's in the same namespace)
and you will see the authentication logic.

As you scroll through the methods (which are in a logical order), you will see
this one:

```php
<?php

    /**
     * Handle a login request to the application.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function login(Request $request)
    {
        $this->validateLogin($request);

        // If the class is using the ThrottlesLogins trait, we can automatically throttle
        // the login attempts for this application. We'll key this by the username and
        // the IP address of the client making these requests into this application.
        $throttles = $this->isUsingThrottlesLoginsTrait();

        if ($throttles && $lockedOut = $this->hasTooManyLoginAttempts($request)) {
            $this->fireLockoutEvent($request);

            return $this->sendLockoutResponse($request);
        }

        $credentials = $this->getCredentials($request);

        if (Auth::guard($this->getGuard())->attempt($credentials, $request->has('remember'))) {
            return $this->handleUserWasAuthenticated($request, $throttles);
        }

        // If the login attempt was unsuccessful we will increment the number of attempts
        // to login and redirect the user back to the login form. Of course, when this
        // user surpasses their maximum number of attempts they will get locked out.
        if ($throttles && ! $lockedOut) {
            $this->incrementLoginAttempts($request);
        }

        return $this->sendFailedLoginResponse($request);
    }
```

The `attempt` call is what we are interested in. It does this:
"Attempt to authenticate a user using the given credentials."

We pass it the user's credentials and whether to remember the user's
session. It currently checks to see if the request has a 'remember' input.

However, we can simply change this to 'true', to always force "remember me".

But we can't make the change in this file, as it's part of the framework. We
have to overwrite this method in our `AuthController`. Copy the whole method to
your `AuthController` and change it to force "remember me".

```php
<?php
if (Auth::guard($this->getGuard())->attempt($credentials, true) {
    return $this->handleUserWasAuthenticated($request, $throttles);
}
```

Now your Laravel application will always remember a user's session, so they
won't have to login again. Unless they clear their cookies.