# Facebook

## Content

- [Requirements - what do you need](#requirements)
- [Installation - how to register an extension](#nstallation)
- [Usage - how to use it](#usage)
- [JavaScript - login button](#javascript)

## Requirements

You need to create a FacebookApp and supply these parameters:

* **appId**
* **appSecret**

## Installation

```yaml
extensions:
    facebook: Contributte\Facebook\DI\Nette\FacebookExtension
    
facebook:
    appId: %yourAppId%
    appSecret: %yourAppSecret%
```

## Usage

Simple example how to use Facebook Login in Presenter

```php
namespace App\Presenters;

use Contributte\Facebook\Exceptions\FacebookLoginException;
use Contributte\Facebook\FacebookLogin;
use Nette\Application\Responses\RedirectResponse;
use Nette\Application\UI\Presenter;
use Nette\Security\AuthenticationException;

final class SignPresenter extends Presenter
{

    /** @var FacebookLogin @inject */
    public $facebook;

    public function actionFacebook()
    {
        // Redirect to FB and ask customer to grant access to his account
        $url = $this->facebook->getLoginurl($this->link('//facebookAuthorize'), ['email', 'public_profile']);
        $this->sendResponse(new RedirectResponse($url));
    }

    /**
     * Log in user with accessToken obtained after redirected from FB
     *
     * @return void
     */
    public function actionFacebookAuthorize()
    {
        // Fetch User data from FB and try to login
        try {
            $token = $this->facebookLogin->getAccessToken();

            $this->user->login('facebook', $this->facebookLogin->getMe($token, ['first_name', 'last_name', 'email', 'gender']));
            $this->flashMessage('Login successful :-).', 'success');
        } catch (FacebookLoginException | AuthenticationException $e) {
            $this->flashMessage('Login failed. :-( Try again.', 'danger');
        }
    }

}

```

## JavaScript

You can also use FB login button, for example:

```
<div 
    class="fb-login-button" 
    onlogin="fbAfterLogin()" 
    data-width="200" 
    data-max-rows="1" 
    data-size="medium" 
    data-button-type="continue_with" 
    data-show-faces="false" 
    data-auto-logout-link="false" 
    data-use-continue-as="true" 
    data-scope="email,public_profile"
>
Login
</div>
```

And use `onlogin` event to call backend code which takes care of registration/login process:

```php
/**
 * Log in user with accessToken from cookie/session after javascript authorization
 */
public function actionFacebookCookie()
{
    // Fetch User data from FB and try to login
    try {
        $token = $this->facebookLogin->getAccessTokenFromCookie();

        $this->user->login('facebook', $this->facebookLogin->getMe($token, ['first_name', 'last_name', 'email', 'gender']));
        $this->flashMessage('Login successful :-).', 'success');
    } catch (FacebookLoginException | AuthenticationException $e) {
        EconeaLogger::fatal($e);
        $this->flashMessage('Login failed. :-( Try again.', 'danger');
    }
}
```
