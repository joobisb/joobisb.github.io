+++
title = 'Troubleshooting Google Digital Asset Links for Seamless Android App Linking'
date = 2023-11-11T00:23:24+05:30
readTime = true
featuredImage = '/images/android-app-links/44817ade-6e15-4d30-9038-25d0fa1772af.png'

+++

We all want to provide a seamless experience for the users of our app. Android app links are a great way to launch users straight into the app when they click on a link.

There are different types of links that you can create in an Android app.

* Deep Links
    
* Web Links
    
* App Links
    

I've been working on a product for which we needed to use app links. Here I'll discuss solving issues with setting up Android app links.

If you are new and implementing app links from scratch, I suggest you look through [android-app-links](https://developer.android.com/training/app-links).

## Add Google digital asset links

In order to use app links we need to prove the ownership of the domain by uploading `assetlinks.json` to `.well-known` directory in the root of the domain.

`https://<mydomain.com>/.well-known/assetlinks.json`

Here is an example of Uber's assetlinks.json [https://uber.com/.well-known/assetlinks.json](https://uber.com/.well-known/assetlinks.json)

Our product was using [Webflow](https://webflow.com/) to host the website and it [did not support](https://discourse.webflow.com/t/add-google-digital-asset-links/98009/3) adding Google digital asset links.

### Setup reverse proxy

The options were either to download the content and host it ourselves or set up a reverse proxy, chose the latter.

Used Nginx for this and the config is as follows:

```nginx
server {
	listen 80 default_server;
	listen [::]:80 default_server;

	root /var/www/html;

	# Add index.php to the list if you are using PHP
	index index.html index.htm index.nginx-debian.html;

	server_name _;
	location /health {
		access_log off;
		return 200 "OK";
		add_header Content-Type text/plain;
	}

	location /.well-known/ {
        alias /var/www/html/.well-known/;
		add_header Content-Type application/json;
    }

	location / {
		
        proxy_pass https://<web.mydomain.com>;
        proxy_busy_buffers_size   512k;
 		proxy_buffers   4 512k;
 		proxy_buffer_size   256k;
		proxy_ssl_server_name on; 
        proxy_set_header  X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	}
}
```

Following are the steps followed to set this up.

1. Changed the configured domain in Webflow from `https://<mydomain.com>` to `https://<web.mydomain.com>` .
    
2. Brought a reverse proxy in between this and served the `https://<mydomain.com>` site through this.
    
3. Copied the `assetlinks.json` file to `/var/www/html/.well-known/` path of the Nginx container while building the Docker image for Nginx.
    
4. Now `assetlinks.json` was available at `https://<mydomain.com>/.well-known/assetlinks.json`
    
5. But when validated using [google-digital-asset-links-tester](https://developers.google.com/digital-asset-links/tools/generator), it failed.
    
    ![](/images/android-app-links/65285640-de2c-48d8-a49c-cdf0d0f485c5.png)
    

When the network was inspected, it was throwing

```
unavailable: Redirect encountered while fetching statements from 
https://<mydomain.com>/.well-known/assetlinks.json (which is 
equivalent to https://<mydomain.com>/.well-known/assetlinks.json): 
redirects are disallowed for security reasons 
(NOT_FOLLOWED_MAX_FORWARDS)
```

> As per google official [doc](https://developers.google.com/digital-asset-links/v1/create-statement#:~:text=The%20assetlinks.,response%20codes%20are%20not%20followed),
> 
> The assetlinks.json file must be served as `Content-Type: application/json` in the HTTP headers, and it cannot be a redirect (that is, `301` or `302` response codes are not followed).

In our case, initially `assetlinks.json` was served via a redirect as shown below.

![](/images/android-app-links/b1088687-d0df-4ee7-bd4c-eb6a7912d273.png)

Since redirects were a security restriction, modified the flow as follows:

![](/images/android-app-links/ecdbbe68-78c7-4f30-9ba1-c4060483d7a9.png)

Initially, we had a redirect in order to handle some use cases for certain paths, in the updated flow this was included in the Nginx conf so that `assetlinks.json` can be served without redirects.

Now validated again using [google-digital-asset-links-tester](https://developers.google.com/digital-asset-links/tools/generator), my domain was able to grant deep linking permission to my app.

![](/images/android-app-links/e385ad55-702d-4d13-bcc8-cebd3e9b77a9.png)

Let me know if you have faced similar issues with Android app links in the comments.

## References

* [https://developer.android.com/training/app-links](https://developer.android.com/training/app-links)
    
* [https://developers.google.com/digital-asset-links/v1/create-statement](https://developers.google.com/digital-asset-links/v1/create-statement)

---
*This blog was originally published [here](https://blog.joobisb.com/solving-issues-with-setting-up-google-digital-asset-links-for-android-app-links)*