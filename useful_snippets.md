### Redirect user from controller

As you search for this stuff in google you'll find a lot of different and bad examples of redirecting. Note that redirecting from form and from controller are two different things. So if you have a custom routing and controller and you want to redirect a user to a page just use this snippet:

```php
use Symfony\Component\HttpFoundation\RedirectResponse;

$url = Url::fromRoute('cp_authentication.login');
$response = new RedirectResponse($url->toString());
$response->send();
```
