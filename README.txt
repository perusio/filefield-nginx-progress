// $Id:

The FileField Nginx Progress module expands FileField's javascript upload progress bar to work with the nginx webserver (when compiled with the upload progress module).

FileField Nginx Progress was written and is maintained by Ben Osman (smoothify).
- http://radiantflow.com


Required Drupal Modules
--------------------------

* FileField                   - http://drupal.org/project/filefield


Required Nginx Modules 
--------------------------

* Upload Progress Module      - http://github.com/masterzen/nginx-upload-progress-module


Module Usage
--------------------------

This module currently has no configuration options, simply install it in drupal and it will add the upload progress functionality to all FileFields and ImageFields.

However, for it to work your nginx must be set up correctly.



Nginx Configuration
--------------------------

You must be running the nginx webserver compiled with MasterZen's Upload Progress module. 

Then, you must configure nginx.conf to track upload progress correctly (the ... are your existing configuration) :

http {

    ... 

    # Upload Progress: reserve 1MB under the name 'uploads' to track uploads
    upload_progress uploads 1m;

    server {

        ...

        location ~ (.*)/x-progress-id:(\w*) {
            rewrite ^(.*)/x-progress-id:(\w*)  $1?X-Progress-ID=$2;
        }

        location ^~ /progress {
            report_uploads uploads;
        }


        ...

        #if you are using php-fpm or fast_cgi
        location ~ \.php$ {

            ...

            fastcgi_index index.php;

            track_uploads uploads 60s;

        } 

        #or if you have nginx acting as a proxy
        location ~ \.php$ {

            ...

            proxy_redirect default;

            track_uploads uploads 60s;

        } 
    


    }


More documentation on configuration is available here:
http://github.com/masterzen/nginx-upload-progress-module/blob/master/README
http://wiki.nginx.org/NginxHttpUploadProgressModule


Frequently Asked Questions
--------------------------

Q) How do I compile nginx with the upload progress module?

A) There is basic information here: http://wiki.nginx.org/NginxHttpUploadProgressModule#Installation
