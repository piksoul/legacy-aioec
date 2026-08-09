# legacy-aioec

Locally patched fork of **All-in-One Event Calendar by Time.ly**, pinned at
the last version (3.0.1) that still runs a fully standalone calendar engine
without requiring a Time.ly cloud account. Versions from 3.0.0 onward route
new installs through a cloud-only ("apiki") bootstrap with no local
fallback; this site was already running the legacy engine before that
change and stays on it deliberately.

## Why this fork exists

The upstream plugin is effectively abandoned for the standalone use case:
no PHP 8 compatibility work has been done since a PHP 7.4 cleanup pass, and
the vendor's own update channels would happily push the cloud-only variant
onto this site if left enabled. See `source/all-in-one-event-calendar/`
for the patched plugin source, and the "LOCAL PATCH NOTES" comment at the
top of `all-in-one-event-calendar.php` for a summary maintained alongside
the code.

## Patches applied (local version 3.0.1.1)

1. **PHP 8 compatibility** — fixed fatal errors and deprecation notices in
   the standalone engine and its bundled vendor libraries (Twig 1.x,
   lessphp, iCalcreator): removed `create_function()`, fixed optional-
   before-required and implicit-nullable parameter deprecations, fixed a
   `preg_match()` null-flags deprecation, and satisfied `Countable`/
   `IteratorAggregate` return-type requirements. Verified with `php -l`
   across all source files and a live `Twig_Environment` render under
   PHP 8.4.
2. **Early textdomain-load notice** — deferred a translation call in
   `Ai1ec_Front_Controller` that was running before WordPress's `init`
   hook (a WP 6.7+ `doing_it_wrong` notice), which was firing on every
   request due to a persistent legacy theme-setting condition.
3. **Update checks disabled** — added `Update URI: false` to the plugin
   header (opts out of the wordpress.org update check) and unregistered
   the plugin's own `pre_set_site_transient_update_plugins` / `plugins_api`
   filters in `app/controller/front.php`, which otherwise poll
   `update.time.ly` / `checkout.time.ly` directly and can inject an
   "update available" prompt into WordPress's normal update UI
   independent of wordpress.org. This prevents either channel from
   silently replacing this patched build with a stock or cloud-forced
   release.

Do not run "Update Now" on this plugin, and do not accept a Time.ly
update prompt, without first checking whether these patches still apply
to whatever version is being offered.
