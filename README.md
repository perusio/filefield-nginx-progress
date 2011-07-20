# Filefield Nginx Progress

## Introduction 

This module provides support for an upload progress bar for a
backend/server implementing the
[RFC 1867](http://www.faqs.org/rfcs/rfc1867.html) upload using
multipart/form-data. Nginx does not yet support RFC 1867 uploads in
its suite of official modules. You can use the
[Upload](https://github.com/vkholodkov/nginx-upload-module) 3rd party
module to do that or just rely on PHP FastCGI support for RFC 1867.

## Drupal Requirements

 * For **drupal 6** the
   [filefield module](http://drupal.org/project/filefield) must be installed.
      
 * For **drupal 7** the file field is now a
   [core module](http://api.drupal.org/api/drupal/modules--file--file.module/7)
   and must be enabled. 

## Nginx Requirements 

 1. Using Nginx with a FastCGI backend, making use of the
    [FastCGI](http://wiki.nginx.org/HttpFcgiModule) module with the
    PHP backend.
    or using Nginx as a reverse proxy, making use of the
    [Proxy](http://wiki.nginx.org/HttpProxyModule) module.
  
 2. Use the 3rd party 
    [Upload Progress](https://github.com/masterzen/nginx-upload-progress-module)
    Nginx module.
    
    Build instructions are
    [here](http://wiki.nginx.org/HttpUploadProgressModule#Installation).
    
    Verify that the upload progress module is installed by issuing:
        
        nginx -V
        
    and check if there's a line with
    `--add-module=path/to/nginx-upload-progress-module`. If there is
    that means that the support for the upload progress bar is
    compiled in. Now you need to support the upload progress bar in
    the Nginx configuration.
    
## Instalation 

 1. Install the module as usual from
    [drupal.org](https://drupal.org/project/filefield_nginx_progress).
    
 2. Add the support for the upload progress bar in your nginx
    configuration.
    
 3. Done.
 
## Nginx configuration

The configuration consists in three parts:

 1. Add a **zone** for **tracking** the uploads. This is a memory block
    used for storing the information regarding the upload that Nginx is
    proxying.
   
        ## Define a zone named uploads with a 1MB size.
        upload_progress uploads 1m;
       
    This is to be done at the `http` level of your config, meaning
    that it's the same for **all** vhosts configured on an instance of
    Nginx.
    
 2. Do a rewrite to support the way that the Nginx module reports the
    progress.
   
        ## The Nginx module wants ?X-Progress-ID query parameter so
        ## that it report the progress of the upload through a GET
        ## request. But the drupal form element makes use of clean
        ## URLs in the POST.
        location ~ (.*)/x-progress-id:(\w*) {
           rewrite ^(.*)/x-progress-id:(\w*)  $1?X-Progress-ID=$2;
        }

        ## Now the above rewrite must be matched by a location that
        ## activates it and references the above defined upload
        ## tracking zone.
        locate ^~ /progress {
           report_uploads uploads;
        }
 
 3. Now on **each** location where you want the upload progress bar
    to work you must enable it like on this example.
        
        location = /index.php {
           include fastcgi.conf;
           fastcgi_pass phpcgi;
           ## Track uploads for this location on the zone defined
           ## above with a 60 seconds timeout.
           track_uploads uploads 60s;
        }
         
 4. Reload your nginx configuration:
 
        service nginx reload
     
 5. Done.    
     
 
## TODO
 
 1. Use the `X-Progress-ID` header instead of issuing a GET request
    and relying on the above rewrite. It will make the configuration
    easier to understand and faster.
     
 2. Make the 7.x version work correctly. Currently there are some
    problems with it as can be seen on the issue queue.
     
## Credits & Acknowledgments

This module development is done by
[smoothify](http://drupal.org/user/115335) sponsored by the
[Spirit Library](http://spiritlibrary.com) and
[perusio](http://drupal.org/user/8859) &mdash; focusing on the D7 version.
