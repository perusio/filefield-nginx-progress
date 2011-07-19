// $Id:

The FileField Nginx Progress module expands the File field's javascript
upload progress bar to work with the nginx webserver (when compiled
with the upload progress module).

FileField Nginx Progress was written and is maintained by Ben Osman (smoothify).
- http://radiantflow.com


Required Drupal Modules
--------------------------

* File                        - Part of Drupal 7 Core


Required Nginx Modules 
--------------------------

* Upload Progress Module      - http://github.com/masterzen/nginx-upload-progress-module


Module Usage
--------------------------

This module currently has no configuration options, simply install it
in drupal and it will add the upload progress functionality to all
File and Image fields.

However, for it to work your nginx must be set up correctly.


Status Report
---------------------------

The module checks to see if you have a PHP version later than
5.2.0. There's no need for APC or the PECL uploadprogress extensions:
nginx makes no use of them. The module reports in the status report page
admin/reports/status the upload progress bar capability.

Note that the nginx_upload_progress module must be compiled in
nginx. You can check that by issuing /usr/sbin/nginx -V or whatever
invoking nginx with the switch -V in whatever location it's installed
in your system.

It will tell you if add_module has a setting of
nginx-upload-progress. If so then your server supports upload
progress. The module cannot run the command nginx -V to find out if
the upload progress module is enabled. Usually web servers run as an
unprivileged user for obvious security reasons. Therefore it defaults
to assuming that nginx_upload_progress is compiled in. It's your
responsibility to ensure that it is so.


Nginx Configuration
--------------------------

You must be running the nginx webserver compiled with MasterZen's
Upload Progress module.

Then, you must configure nginx.conf to track upload progress correctly
(the ... are your existing configuration) :

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

A) There is basic information here:
   http://wiki.nginx.org/NginxHttpUploadProgressModule#Installation
