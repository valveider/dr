dr
==
server {
        server_name domain.tld;
        root /var/www/drupal6; ## <-- Your only path reference.
 
        location = /favicon.ico {
                log_not_found off;
                access_log off;
        }
 
        location = /robots.txt {
                allow all;
                log_not_found off;
                access_log off;
        }
 
        # This matters if you use drush prior to 5.x
        # After 5.x backups are stored outside the Drupal install.
        location = /backup {
                deny all;
        }
 
        # Very rarely should these ever be accessed outside of your lan
        location ~* \.(txt|log)$ {
                allow 192.168.0.0/16;
                deny all;
        }
 
        location ~ \..*/.*\.php$ {
                return 403;
        }
 
        # No no for private
        location ~ ^/sites/.*/private/ {
                return 403;
        }
 
        # Block access to "hidden" files and directories whose names begin with a
        # period. This includes directories used by version control systems such
        # as Subversion or Git to store control files.
        location ~ (^|/)\. {
                return 403;
        }
 
        location / {
                # This is cool because no php is touched for static content
                try_files $uri @rewrite;
        }
 
        location @rewrite {
                # You have 2 options here
                # For D7 and above:
                # Clean URLs are handled in drupal_environment_initialize().
                rewrite ^ /index.php;
                # For Drupal 6 and bwlow:
                # Some modules enforce no slash (/) at the end of the URL
                # Else this rewrite block wouldn't be needed (GlobalRedirect)
                # rewrite ^/(.*)$ /index.php?q=$1;
        }
 
        location ~ \.php$ {
                fastcgi_split_path_info ^(.+\.php)(/.+)$;
                #NOTE: You should have "cgi.fix_pathinfo = 0;" in php.ini
                include fastcgi_params;
                fastcgi_param SCRIPT_FILENAME $request_filename;
                fastcgi_intercept_errors on;
                fastcgi_pass unix:/tmp/phpfpm.sock;
        }
 
        # Fighting with ImageCache? This little gem is amazing.
        location ~ ^/sites/.*/files/imagecache/ {
                try_files $uri @rewrite;
        }
        # Catch image styles for D7 too.
        location ~ ^/sites/.*/files/styles/ {
                try_files $uri @rewrite;
        }
 
        location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
                expires max;
                log_not_found off;
        }
}
