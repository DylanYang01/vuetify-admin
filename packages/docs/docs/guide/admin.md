# Admin

### Instanciation

In order to operate, VtecAdmin constructor needs all of this options :

* Vue Router instance, which can contains all your custom routes, for automatic resource URL Crud pages registering.
* Vue Store instance, which can contains all your custom modules, for automatic resource API modules bridge registering.
* Vue I18n instance, which can contains all your custom localized labels, for full internationalization support.
* Title of your admin app.
* At least one provided locale, only `en` and `fr` are 100% supported, but you can easily add your own by following [this model](src/locales/fr.json).
* Auth and data providers (see next section).
* Optional file browser URL, which will appear on included TinyMCE file picker.
* A resources array which contain all resources to administer. Each resource can have :
  * A mandatory slug name which will be used for api url calls.
  * An icon for identify it in sidebar or list page.
  * Indicator if this resource can be translated, in this case, a new query string with locale will be added on each api calls, it's up to you to handle it on backend.
  * List of available actions for this resource. By default all 5 operations are active (list / show / create / edit / delete). You can use `except` or `only` properties for blacklist or whitelist few actions. All removed actions will reflected on all crud pages and Vue Router will be adapted accordingly.
  * Permissions for user resource operations filtering.

> Check code sample on [main readme](https://github.com/okami101/vtec-admin#code)

## Included auth and data Providers

* [Laravel Sanctum auth provider](src/providers/sanctumAuthProvider.js) which use good old full state cookies authentication. You are free to replace it by your own provider by implementing [auth actions methods](src/utils/authActions.js). JWT is a more common way for SPA app but is also sensitive to XSS attacks, contrary to standard cookies if set to http only. If admin app is on the same main domain, it's often preferable to stay with cookies. Besides, this is by far the easiest way to get impersonation and elFinder (best known of PHP files manager) working as-is. If you use JWT, you can forget about them...
* [Laravel data provider](src/providers/laravelDataProvider.js) which has full compatibility with Laravel resource API and work seamlessly with [Spatie query builder](https://github.com/spatie/laravel-query-builder), the ideal package for browsing resources. But as this package is intended to be fully independent of any backends, you can obviously use your own provider for any type of backend by implementing [data actions methods](src/utils/dataActions.js) which allow full compatibility with Vtec Admin.

> Both included providers needs [axios](https://github.com/axios/axios) in order to work.