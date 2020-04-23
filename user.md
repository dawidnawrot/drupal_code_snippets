# User snippets

## User is logged in
```php
// Check if user is logged in.
if (\Drupal::currentUser()->isAuthenticated()) {
  // User is logged in.
}

// Check if user is not logged in.
if (\Drupal::currentUser()->isAnonymous()) {
  // User is not logged in. 
}
```
