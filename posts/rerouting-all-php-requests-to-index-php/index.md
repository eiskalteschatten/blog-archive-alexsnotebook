<figure><img loading="lazy" decoding="async" src="php-logo-transparent-6-e1694523486642-940x565.png" alt="PHP"></figure>

A common need for a lot of PHP applications is the ability to have the entire application run through an index.php file, but with clean URLs. While this post won’t go into how to do that or why you would want to, it will provide the configuration for Apache that will allow you to do it:

```apacheconf
<IfModule mod_rewrite.c>
    # Redirect /index.php to / (optional, but recommended I guess)
    RewriteCond %{THE_REQUEST} ^[A-Z]{3,9}\ /.*index\.php
    RewriteRule ^index.php/?(.*)$ $1 [R=301,L]

    # Run everything else but real files through index.php
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^(.*)$ index.php/$1?%{QUERY_STRING} [L]
</IfModule>
```

This can either go into the .htaccess file at the root of your project or in the virtual host configuration file.