# WebP on demand

This is a solution for automatically serving WebP images instead of jpeg/pngs for browsers that supports WebP (Google Chrome, that is).

Once set up, it will automatically convert images, no matter how they are referenced. It for example also works on images referenced in CSS. As the solution does not require any change in the HTML, it can easily be integrated into any website / framework (A Wordpress adaptation was recently published on [wordpress.org](https://wordpress.org/plugins/webp-express/) - its also on [github](https://github.com/rosell-dk/webp-express))

WebP on demand consists of two parts.

- *The redirect rules* detects whether a requested image has already been converted. If so, it redirects it to the converted image. If not, it redirects it to the *image converter*

- *The image converter* converts, saves and serves the image. We are using the [webp-convert-and-serve](https://github.com/rosell-dk/webp-convert-and-serve) library for this.

** New feature: LiteSpeed support **
The redirect rules are written for Apache. They did not work on *LiteSpeed* servers (even though LiteSpeed claims compliance). A little tweaking however solved the problem.

This project currently only supports Apache and LiteSpeed. In time, we may add redirect rules for NGINX and/or Windows Server. If you don't want to wait for NGINX support, there are NGINX rules to get you started [here](https://github.com/S1SYPHOS/kirby-webp#nginx).


## Installation

#### 1. Clone or download this repository
#### 2. Install dependencies
- Run composer: `composer install`

#### 3. Get `webp-on-demand.php` up and running
1. Copy the supplied `webp-on-demand.php.example` into your website - wherever you like - removing '.example' from filename

2. Unless you have placed `webp-on-demand.php` in webroot, you must alter it a bit. If for example, you have placed it in a subfolder (1 level deep), first line should be altered to:
```
require '../webp-on-demand/vendor/autoload.php';
```

3. Test that the converter is working. Place an image in your root and point your browser to `http://your-domain.com/your-folder/webp-on-demand.php?source=your-image.jpg&debug`.

The debug parameter causes the script to return text instead of the converted image.
You should now see a report on how the conversion went.

If no converters are available, you can try to make a converter work or hook up to a cloud converter. You can learn more about available options at the github page for [webp-convert](https://github.com/rosell-dk/webp-convert)

#### 4. Get the `.htaccess` file up and running

Decide what part of your website you want WebPOnDemand to work on. The rewrite rules must be placed in a .htaccess file in that directory. You get the rewrite rules from the `.htaccess.template` file in this repository. It is called ".template" to signify that you must replace some placeholders in order to get the working rules. Instructions are placed in `.htaccess.template` itself.

To test that the `.htaccess` is routing your image to the image converter, point your browser to `http://your-domain.com/your-folder/your-image.jpg&debug`. If you see a textual report, the redirect is working. If you see an image, it is not working. (the `.htaccess` rules are set up to forward the querystring, so - if things are working correctly - webp-on-demand.php will be called with "?debug", and therefore produce a textual report)

In order to test that a image is not being reconverted every time it is requested, check the Response headers of the image. There should be a "X-WebP-Express" header. It should say Routed to image converter" the first time, but "Routed to existing converted image" on subsequent requests.
To view response headers in Google Chrome do the following:

- Open the image in Google Chrome (point it to `http://your-domain.com/your-folder/your-image.jpg`)
- Right-click the image and choose "Inspect"
- Click the "Network" tab
- Reload the page
- Click the image in the list, you should now see a list of headers



#### 5. Use it!
You do not have to make any changes to your existing HTML or CSS. The routing and conversion are now done automatically. To confirm that it works:

1. Visit a page on your site with an image on it, using *Google Chrome*.
- Right-click the page and choose "Inspect"
- Click the "Network" tab
- Reload the page
- Find a jpeg or png image in the list. In the "type" column, it should say "webp"


## Troubleshooting the `.htaccess` file.
By appending `?debug` to your image url, you get a report (text output) instead of the converted image. If there is no report, it means that the `.htaccess` is not working as intended.

By appending `?reconvert` to your image url, you bypass the automatic routing to existing source files.

As described in the install section, you can inspect 'X-WebP-On-Demand' header to detect which rewrite rules are triggered, if any.

Is something not working?
- Perhaps there are other rules in your `.htaccess` that interfere with the rules?
- Perhaps your site is on Apache, but it has been configured to use *Nginx* to serve image files. You then need to reconfigure your server setup. Or create Nginx rules. There are some [here](https://github.com/S1SYPHOS/kirby-webp#nginx).


## `webp-on-demand.php` options.

You add options to `webp-on-demand.php` directly in the `.htaccess`. You can however also add them after an image url.

| option                         | Description                                          |
| ------------------------------ | --------------------------------------------- |
| *base-path*                    | Sets the base path used for "source" and "destination-root" options. Must be relative to document root, or absolute (not recommended). When used in .htaccess, set it to the folder containing the .htaccess file, relative to document root. If for example document root is /var/www/example.com/ and you have a subdirectory "wordpress", which you want WebPOnDemand to work on, you should place .htaccess rules in the "wordpress" directory, and your "base-path" will be "wordpress". If not set, it defaults to be the path of webp-on-demand.php |
| *source*                       | Path to source file, relative to 'base-path' option. The final path is calculated like this: [base-path] + [path to source file] + ".webp". Absolute path is depreciated, but supported for backwards compatability.|
| *destination-root* (optional)  | The path of where you want the converted files to reside, relative to the 'base-path' option. If you want converted files to be put in the same folder as the originals, you can set destination-root to ".", or leave it blank. If you on the other hand want all converted files to reside in their own folder, set the destination-root to point to that folder. The converted files will be stored in a hierarchy that matches the source files. With destination-root set to "webp-cache", the source file "images/2017/cool.jpg" will be stored at "webp-cache/images/2017/cool.jpg.webp". Double-dots in paths are allowed, ie "../webp-cache". The final destination is calculated like this: [base-path] + [destination-root] + [path to source file] + ".webp". Default is ".". You can also supply an absolute path|
| *quality* (optional)           | The quality of the generated WebP image, "auto" or 0-100. Defaults to "auto". See [WebPConvert](https://github.com/rosell-dk/webp-convert#methods) docs |
| *max-quality* (optional)        | The maximum quality. Only relevant when quality is set to "auto" |
| *default-quality* (optional)    | Fallback value for quality, if it isn't possible to detect quality of jpeg. Only relevant when quality is set to "auto" |
| *metadata* (optional)          | If set to "none", all metadata will be stripped. If set to "all", all metadata will be preserved. See [WebPConvert](https://github.com/rosell-dk/webp-convert#methods) docs |
| *converters* (optional)        | Comma-separated list of converters. Ie. "cwebp,gd". To pass options to the individual converters, see next. Also, check out the [WebPConvert](https://github.com/rosell-dk/webp-convert#methods) docs |
| *[converter-id]-[option-name]* (optional)  | This pattern is used for setting options on the individual converters. Ie, in order to set the "key" option of the "ewww" converter, you pass "ewww-key".
| *[converter-id]-[n]-[option-name]* (optional)  | Use this pattern for targeting options of a converter, that are used multiple times. However, use the pattern above for targeting the first occurence. `n` stands for the nth occurence of that converter in the `converters` option. Example: `...&converters=cwebp,ewww,ewww,gd,ewww&ewww-key=xxx&ewww-2-key=yyy&ewww-3-key=zzz&gd-skip-pngs=1` |
| *[converter-id]-[option-name]-[2]* (optional) | This is an alternative, and simpler pattern than the above, for providing fallback for a single converter. If WebPOnDemand detects that such an option is provided (ie ewww-key-2=yyy), it will automatically insert an extra converter into the array (immidiately after), configured with the options with the '-2' postfix. Example: `...&converters=cwebp,ewww,gd&ewww-key=xxx&ewww-key-2=yyy` - will result in converter order: cwebp, ewww (with key=xxx), ewww (with key=yyy), gd |
| *debug* (optional)             | If set, a report will be served (as text) instead of an image |
| *fail* (optional)              | What to serve if conversion fails. Default is  "original". Possible values: "original", "404", "report", "report-as-image". See [WebPConvertAndServe](https://github.com/rosell-dk/webp-convert-and-serve#api) docs|
| *critical-fail* (optional)              | What to serve if conversion fails and source image is not availabl Default is  "error-as-image". Possible values: "original", "404", "report", "report-as-image". See [WebPConvertAndServe](https://github.com/rosell-dk/webp-convert-and-serve#api) docs |


## Configuring the converter

You configure the options to the image converter directly in the ```.htaccess```.

If you want to have a different quality on a certain image, you can append "&reconvert&quality=95" to the image url. You can in fact override any change any converter option like this.

You configure the options to the image converter directly in the ```.htaccess```.

If you want to have a different quality on a certain image, you can append "&reconvert&quality=95" to the image url. You can in fact override any change any converter option like this.


## Requirements

* Apache web server (not tested on LiteSpeed yet)
* PHP > 5.5.0
* That one of the WebP converters are working (these have different requirements)


## FAQ

### How do I make this work with a CDN?
Chances are that the default setting of your CDN is not to forward any headers to your origin server. But the solution needs the "Accept" header, because this is where the information is whether the browser accepts webp images or not. You will therefore have to make sure to configure your CDN to forward the "Accept" header.

The .htaccess takes care of setting the "Vary" HTTP header to "Accept" when routing WebP images. When the CDN sees this, it knows that the response varies, depending on the "Accept" header. The CDN is thus instructed not to cache the response on URL only, but also on the "Accept" header. This means that it will store an image for every accept header it meets. Luckily, there are (not that many variants for images[https://developer.mozilla.org/en-US/docs/Web/HTTP/Content_negotiation/List_of_default_Accept_values#Values_for_an_image], so it is not an issue.


## Detailed explanation of how it works

### Workflow:

1. The .htaccess routes convert request to *webp-on-demand.php*, with options in query string
2. *webp-on-demand.php* basically just calls `WebPOnDemand::serve(__DIR__)`
3. `WebPOnDemand::serve` routes the options to  [WebPConvertAndServe](https://github.com/rosell-dk/webp-convert-and-serve)
4. `WebPConvertAndServe` in turn delegates the conversion to [WebPConvert](https://github.com/rosell-dk/webp-convert)

### The Apache configuration files in details

Lets make a walk-through of .htaccess-template.

The rules read:
```
<IfModule mod_rewrite.c>

    RewriteEngine On

    # Redirect to existing converted image (under appropriate circumstances)
    RewriteCond %{HTTP_ACCEPT} image/webp
    RewriteCond %{QUERY_STRING} !((^reconvert.*)|(^debug.*))
    RewriteCond %{DOCUMENT_ROOT}/your-base-path/your-destination-path/$1.$2.webp -f
    RewriteRule ^\/?(.*)\.(jpe?g|png)$ /your-base-path/your-destination-path/$1.$2.webp [NC,T=image/webp,E=WEBPACCEPT:1,E=WEBPEXISTING:1,QSD]

    # Redirect to converter (under appropriate circumstances)
    RewriteCond %{HTTP_ACCEPT} image/webp
    RewriteCond %{QUERY_STRING} (^reconvert.*)|(^debug.*) [OR]
    RewriteCond %{DOCUMENT_ROOT}/your-base-path/your-destination-path/$1.$2.webp !-f
    RewriteCond %{QUERY_STRING} (.*)
    RewriteRule ^\/?(.*)\.(jpe?g|png)$ your-webp-on-demand-path/webp-on-demand.php?base-path=your-base-path&destination-root=your-destination-path&source=$1.$2&quality=80&fail=original&critical-fail=report&%1 [NC,E=WEBPACCEPT:1,E=WEBPNEW:1]

</IfModule>

<IfModule mod_headers.c>
    # Apache appends "REDIRECT_" in front of the environment variables, but LiteSpeed does not.
    # These next three lines are for Apache, in order to set environment variables without "REDIRECT_"
    SetEnvIf REDIRECT_WEBPACCEPT 1 WEBPACCEPT=1
    SetEnvIf REDIRECT_WEBPEXISTING 1 WEBPEXISTING=1
    SetEnvIf REDIRECT_WEBPNEW 1 WEBPNEW=1

    # Make CDN caching possible.
    # The effect is that the CDN will cache both the webp image and the jpeg/png image and return the proper
    # image to the proper clients (for this to work, make sure to set up CDN to forward the "Accept" header)
    Header append Vary Accept env=WEBPACCEPT

    # Add headers for debugging
    Header append X-WebP-On-Demand "Routed to existing converted image" env=WEBPEXISTING
    Header append X-WebP-On-Demand "Routed to image converter" env=WEBPNEW
</IfModule>
```

First thing to notice is that the code is divided in two blocks. The first redirects to an existing converted image (under appropriate conditions), the second redirects to the image converter (under appropriate conditions)

Also, the blocks only kick in, if the browser supports webp. And there are also lines ensuring that if the image is called with a "debug" or "reconvert" parameter, it will redirect to the converter, rather than to an existing image. The two set of conditions are created such that they are mutual exclusive.

Lets break it down.

#### Redirecting to existing image

First block reads:
```
# Redirect to existing converted image (under appropriate circumstances)
RewriteCond %{HTTP_ACCEPT} image/webp
RewriteCond %{QUERY_STRING} !((^reconvert.*)|(^debug.*))
RewriteCond %{DOCUMENT_ROOT}/your-base-path/your-destination-path/$1.$2.webp -f
RewriteRule ^\/?(.*)\.(jpe?g|png)$ /your-base-path/your-destination-path/$1.$2.webp [NC,T=image/webp,E=WEBPACCEPT:1,E=WEBPEXISTING:1,QSD]
```

*Lets take it line by line:*

`RewriteCond %{HTTP_ACCEPT} image/webp`
This makes sure that the following rule only kicks in when the browser supports webp images. Browsers supporting webp images are obliged to sending a HTTP_ACCEPT header containing "image/webp", which we test for here.

`RewriteCond %{QUERY_STRING} !((^reconvert.*)|(^debug.*))`
This makes sure that the query string does not begin with "reconvert" or "debug" (we want those requests to be redirected to the converter, even when a converted file exists)

`RewriteCond %{DOCUMENT_ROOT}/$1.$2.webp -f`
This makes sure there is an existing converted image. The $1 and $2 refers to matches of the following rule. You may think it is weird that we can reference matches in a rule not run yet, in a condition to that very rule. I agree - mod_rewrite is a complex beast.

`RewriteRule ^\/?(.*)\.(jpe?g|png)$ /your-base-path/your-destination-path/$1.$2.webp [NC,T=image/webp,E=WEBPACCEPT:1,E=WEBPEXISTING:1,QSD]`
Rewrites any request that ends with ".jpg", ".jpeg" or ".png" (case insensitive). The first parentheses makes grabs the file path, which can then be referenced with $1. The second parentheses grabs the file extension into $2. These referenced are used in the preceding condition as well as in the rule itself. The effect of the rewrite is that the target is set to the same as source, but with ".webp" appended to it. Also, MIME type of the response is set to "image/webp" (not necessary, though). The first E flag part sets the environment variable "WEBPACCEPT" to 1. This is used further down in the .htaccess to conditionally append a Vary header. So setting this variable means that the Vary header will be appended if the rule is triggered. The second E sets the environment variable "WEBPEXISTING" to 1. This marks that the request was routed to an existing image and is used later to send a custom header X-WebP-On-Demand="Routed to existing converted image" (practical for debugging). The NC flag makes the match case insensitive. The QSD flag tells Apache to strip the query string. The "\/?" is added to the beginning in order to support LiteSpeed web servers. "your-base-path" and "your-destination-path" must be altered to match the locations of your setup, as instructed in top of `.htaccess.template`

#### Redirecting to image converter
Second block redirects to *the image converter* (under appropiate circumstances)

```
# Redirect to converter (under appropriate circumstances)
RewriteCond %{HTTP_ACCEPT} image/webp
RewriteCond %{QUERY_STRING} (^reconvert.*)|(^debug.*) [OR]
RewriteCond %{DOCUMENT_ROOT}/your-base-path/your-destination-path/$1.$2.webp !-f
RewriteCond %{QUERY_STRING} (.*)
RewriteRule ^\/?(.*)\.(jpe?g|png)$ your-webp-on-demand-path/webp-on-demand.php?base-path=your-base-path&destination-root=your-destination-path&source=$1.$2&quality=80&fail=original&critical-fail=report&%1 [NC,E=WEBPACCEPT:1,E=WEBPNEW:1]
```

*Lets take it line by line*:
`RewriteCond %{HTTP_ACCEPT} image/webp`
We have covered this...

`RewriteCond %{QUERY_STRING} (^reconvert.*)|(^debug.*) [OR]`
If query string contains a "reconvert" or "debug", the block will be activated, even if there exists a converted file. Notice the "[OR]". Know that OR has higher precedence than AND [[ref](https://stackoverflow.com/questions/922399/how-to-use-and-or-for-rewritecond-on-apache)].

`RewriteCond %{DOCUMENT_ROOT}/$1.$2.webp !-f`
Make sure there aren't an existing image (OR above condition)

`RewriteCond %{QUERY_STRING} (.*)`
This is always true. The condition is there to enable us to pass on the querystring from image request to the converter in the next rule, where it will be accessible as "%1"

`RewriteRule ^\/?(.*)\.(jpe?g|png)$ your-webp-on-demand-path/webp-on-demand.php?base-path=your-base-path&destination-root=your-destination-path&source=$1.$2&quality=80&fail=original&critical-fail=report&%1 [NC,E=WEBPACCEPT:1,E=WEBPNEW:1]
`
This line rewrites any request that ends with ".jpg", ".jpeg" or ".png" (case insensitive) to the image converter. You can remove "|png" from the line, if you do not want to convert png files. We have covered the NC flag and the E=WEBPACCEPT in the first block. Notice that we have no "T=image/webp" in this rewrite. It was originally there, but I removed it in order for it to work in LiteSpeed. webp-on-demand.php sets a Content Type header, so it was not needed in the first place. Well, actually, webp-on-demand.php may set content type to text/html (when returning error report), or image/gif, when returning error report as image - so setting it to something different here, is asking for trouble. The "\/?" in the beginning was also added for LiteSpeed support. %1 prints the query string fetched in the preceding condition. This enables overiding the converter options set here. For example appending "&debug&preferred-converters=gd" to an url that points to an image can be used to test the gd converter. Or "&reconvert&quality=100" can be appended in order to reconvert the image using extreme quality. The "E=WEBPNEW:1" flag sets the environment variable "WEBPNEW" to 1. This marks that the request was routed to an existing image and is used later to send a custom header X-WebP-On-Demand="Routed to image converter" (practical for debugging). "your-webp-on-demand-path", "your-base-path" and "your-destination-path" must be altered to match the locations of your setup, as instructed in top of `.htaccess.template`

#### Headers / dealing with CDN

The final part of the .htaccess reads:
```
<IfModule mod_headers.c>
    # Apache appends "REDIRECT_" in front of the environment variables, but LiteSpeed does not.
    # These next three lines are for Apache, in order to set environment variables without "REDIRECT_"
    SetEnvIf REDIRECT_WEBPACCEPT 1 WEBPACCEPT=1
    SetEnvIf REDIRECT_WEBPEXISTING 1 WEBPEXISTING=1
    SetEnvIf REDIRECT_WEBPNEW 1 WEBPNEW=1

    # Make CDN caching possible.
    # The effect is that the CDN will cache both the webp image and the jpeg/png image and return the proper
    # image to the proper clients (for this to work, make sure to set up CDN to forward the "Accept" header)
    Header append Vary Accept env=WEBPACCEPT

    # Add headers for debugging
    Header append X-WebP-On-Demand "Routed to existing converted image" env=WEBPEXISTING
    Header append X-WebP-On-Demand "Routed to image converter" env=WEBPNEW
</IfModule>

AddType image/webp .webp
```

This first appends a response header containing: "Vary: Accept", but only when the environment variable "WEBPACCEPT" is set above. env="REDIRECT_WEBPACCEPT" instructs mod_headers only to append the header, when the "WEBPACCEPT" environment variable is set by the "REDIRECT" module (mod_rewrite).

The two other lines sets headers useful for validating that the rules are working as expected.

## A similar project
This project is very similar to [WebP realizer](https://github.com/rosell-dk/webp-realizer). *WebP realizer* assumes that the conditional part is in HTML, like this:
```
<picture>
  <source srcset="images/button.jpg.webp" type="image/webp" />
  <img src="images/button.jpg" />
</picture>
```
And then it automatically generates "image.jpg.webp" for you, the first time it is requested.\
Pros and cons:

- *WebP on demand* works on images referenced in CSS (*WebP realizer* does not)\
- *WebP on demand* requires no change in HTML (*WebP realizer* does)\
- *WebP realizer* works better with CDN's - CDN does not need to cache different versions of the same URL

## Related
* [My original post presenting the solution](https://www.bitwise-it.dk/blog/webp-on-demand)
* [WebP Express](https://github.com/rosell-dk/webp-express). A Wordpress adaptation of the solution.
* https://www.maxcdn.com/blog/how-to-reduce-image-size-with-webp-automagically/
