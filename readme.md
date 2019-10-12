# Sign In With Apple for Laravel

## Supporting This Package

This is an MIT-licensed open source project with its ongoing development made possible by the support of the community. If you'd like to support this, and our other packages, please consider sponsoring us via the button above.

## Requirements

- PHP 7.2+
- Laravel 6.2+
- Socialite 4.2+
- Apple Developer Subscription

## Installation

```sh
composer require genealabs/laravel-sign-in-with-apple
```

## Configuration

1. Create an `App ID` for your website (https://developer.apple.com/account/resources/identifiers/list/bundleId) with the following details:
    - Platform: iOS, tvOS, watchOS (I'm unsure if either choice has an effect for web apps)
    - Description: (something like "example.com app id")
    - Bundle ID (Explicit): com.example.id (or something similar)
    - Check "Sign In With Apple"
2. Create a `Service ID` for your website (https://developer.apple.com/account/resources/identifiers/list/serviceId) with the following details:
    - Description: (something like "example.com service id")
    - Identifier: com.example.service (or something similar)
    - Check "Sign In With Apple"
    - Configure "Sign In With Apple":
        - Primary App Id: (select the primary app id created in step 1)
        - Web Domain: example.com (the domain of your web site)
        - Return URLs: https://example.com/apple-signin (the route pointing to the callback method in your controller)
        - Click "Save".
        - Click the "Edit" button to edit the details of the "Sign In With Apple"
            configuration we just created.
        - If you haven't verified the domain yet, download the verification file,
            upload it to https://example.com/.well-known/apple-developer-domain-association.txt, and then click the "Verify"
            button.
3. Create a `Private Key` for your website (https://developer.apple.com/account/resources/authkeys/list) with the following details:
    - Key Name: 
    - Check "Sign In With Apple"
    - Configure "Sign In With Apple":
        - Primary App ID: (select the primary app id created in step 1)
        - Click "Save"
    - Click "Continue"
    - Click "Register"
    - Click "Download"
    - Rename the downloaded file to `key.txt`
4. Create your app's client secret:
    - Install the JWT Gem:
        ```sh
        sudo gem install jwt
        ```
    
    - Create a temporary file to process the private key:
        ```ruby
        require 'jwt'

        key_file = 'key.txt'
        team_id = ''
        client_id = ''
        key_id = ''

        ecdsa_key = OpenSSL::PKey::EC.new IO.read key_file

        headers = {
        'kid' => key_id
        }

        claims = {
            'iss' => team_id,
            'iat' => Time.now.to_i,
            'exp' => Time.now.to_i + 86400*180,
            'aud' => 'https://appleid.apple.com',
            'sub' => client_id,
        }

        token = JWT.encode claims, ecdsa_key, 'ES256', headers

        puts token
        ```

    - Fill in the following fields:
        - `team_id`: This can be found on the top-right corner when logged into
            your Apple Developer account, right under your name.
        - `client_id`: This is the identifier from the Service Id created in step
            2 above, for example com.example.service
        - `key_id`: This is the identifier of the private key created in step 3
            above.
    - Save the file and run it from the terminal. It will spit out a JWT which is
        your client secret, which you will need to add to your `.env` file in the
        next step.
5. Set the necessary environment variables in your `.env` file:

    ```env
    SIGN_IN_WITH_APPLE_LOGIN="/apple/login/controller/login/action"
    SIGN_IN_WITH_APPLE_REDIRECT="/apple/login/controller/callback/action"
    SIGN_IN_WITH_APPLE_CLIENT_ID="your app's service id as registered with Apple"
    SIGN_IN_WITH_APPLE_CLIENT_SECRET="your app's client secret as calculated in step 4"
    ```

## Implementation

### Login Button

Add the following blade directive to your login page:

```php
@signInWithAppleButton($color, $hasBorder, $type, $borderRadius)
```

| Parameter | Definition |
| --------- | ---------- |
| $color    | String, either "black" or "white. |
| $hasBorder | Boolean, either `true` or `false`. |
| $type      | String, either `"sign-in"` or `"continue"`. |
| $borderRadius | Integer, greater or equal to 0. |

### Controller
This implementation uses Socialite to get the login credentials. The following is an example implementation of the controller:

```php
<?php

namespace App\Http\Controllers\Auth;

use App\Http\Controllers\Controller;
use Laravel\Socialite\Facades\Socialite;

class AppleSigninController extends Controller
{
    public function __construct()
    {
        $this->middleware('guest')->except('logout');
    }

    public function login()
    {
        return Socialite::driver("sign-in-with-apple")
            ->scopes(["name", "email"])
            ->redirect();
    }

    public function callback(Request $request)
    {
        $user = Socialite::driver("sign-in-with-apple")
            ->user();

        // do what you will with the resulting user object to complete the login process, for example save the user, and log the user in.
    }
}
```

----------

#### Credits
1. https://developer.okta.com/blog/2019/06/04/what-the-heck-is-sign-in-with-apple
2. https://developer.apple.com/sign-in-with-apple/get-started

----------

## Commitment to Quality
During package development I try as best as possible to embrace good design and development practices, to help ensure that this package is as good as it can
be. My checklist for package development includes:

-   ✅ Achieve as close to 100% code coverage as possible using unit tests.
-   ✅ Eliminate any issues identified by SensioLabs Insight and Scrutinizer.
-   ✅ Be fully PSR1, PSR2, and PSR4 compliant.
-   ✅ Include comprehensive documentation in README.md.
-   ✅ Provide an up-to-date CHANGELOG.md which adheres to the format outlined
    at <http://keepachangelog.com>.
-   ✅ Have no PHPMD or PHPCS warnings throughout all code.

## Contributing
Please observe and respect all aspects of the included Code of Conduct <https://github.com/GeneaLabs/nova-map-marker-field/blob/master/CODE_OF_CONDUCT.md>.

### Reporting Issues
When reporting issues, please fill out the included template as completely as
possible. Incomplete issues may be ignored or closed if there is not enough
information included to be actionable.

### Submitting Pull Requests
Please review the Contribution Guidelines <https://github.com/GeneaLabs/nova-map-marker-field/blob/master/CONTRIBUTING.md>. Only PRs that meet all criterium will be accepted.

## If you ❤️ open-source software, give the repos you use a ⭐️.
We have included the awesome `symfony/thanks` composer package as a dev dependency. Let your OS package maintainers know you appreciate them by starring the packages you use. Simply run `composer thanks` after installing this package. (And not to worry, since it's a dev-dependency it won't be installed in your live environment.)
