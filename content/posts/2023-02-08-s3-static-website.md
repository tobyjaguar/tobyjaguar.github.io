---
title: "AWS Cloudfront to s3"
date: 2023-02-08
---

## AWS Cloundfront to s3
![title](/assets/posts/s3-static-website/_amos-4N8oM5L7hyM-unsplash.jpeg)
Photo by <a href="https://unsplash.com/@stockphotos_com?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Amos from Stockphotos</a>

### Setting up a static website with s3

This post, yet again, serves as a message in a bottle for a future self who ever needs to set up a static website on s3, in 2023...because all the documentation changes every year and you never know what will work today. So this obviously will not work tomorrow. But it works today.

### Problem: serve a static website in s3

For this endeavour I thought it would be somewhat approachable to setup a static website in an s3 bucket. I heard you could do it, I have seen it advertised, so I assumed it would be easy enough. Aaaand, it isn't, surprise, surpise, thank you AWS.

### Solution

Upload the files to the bucket and enable `static website hosting`. Done! The bucket gives a url that can be accessed via a browser.

So, head over to your DNS provider, and make the CNAME record, and you are done.
Woohoo! Do that and you will quickly realize that s3 only supports http. Soooo, if you don't want your website to look like a total scamfest, you need https...well, easier said then done.

### Problem: how to serve a static website from s3 for real

To do this for real, it is kind of a pain. The following is a guide for my future self when I forget all the little details to make this work.

Assuming the domain is purchased, and purchased through a provider that is not Route 53:

### First => create a new bucket. 

I am unconvinved that this needs to be exactly the name of the domain.

This will serve static html content, so there are a couple setup constraints. One being that this bucket must disable all permissions. You have to unclick that "unblock all public access" box, and it will yet at you mightily! You will also have the red warning triagle forever mocking you.

Next, as mentioned above the "static website hosting" needs to be enabled.

Lastly, the bucket policy needs to be set for all access:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::BUCKETNAME/*"
        }
    ]
}
```

This is the setup for the bucket, but as we will see later, there is an extra permission we are going to setup to only allow access from cloudfront.

### Second => Request a cert
To get https secure connections, we need a certificate. We are going to request one, NOT CREATE ONE, we are going to request one through AWS Certificate Manager. Navigate there, request a new cert, and click DNS validation.

To get the validation to happen, you will need to create a CNAME record with the name and value given in the validation section after the cert is created. This is annoying because AWS includes the domain name and appends a dot at the end of the name. This will eat up about an hour or so of troubleshooting to figure out that you only need the name portion in front of the dot the preceeds the domain name. So just make the name portion of the CNAME record look like what AWS gave you, and leave off the trailing dot, because it won't work and you can't put it on there anyway. Also the value of the CNAME record is a simple copy and paste.

For the name of the cert, make it the apex name and www, meaning the cert covers:

* example.com
* *.example.com

We need the cert validated before we can select it for the cloudfront distribution.

### Third => create a cloudfront distribution

We want https to work on this website, so we don't look lame. Because s3 only supports http, we need to send all our requests through cloudfront, which will upgrade any request to https and then forward that request to our bucket.

Create a new cloudfront distribution.

For the Orgin domain you want the http reference to the website hosting bucket, not the bucket name itself. If suggested this for me, when I set it up, and said yes, thank you. (So not the Bucket arn but the website endpoint that can be retrieved from the bucket section for website hosting).

We want to add a custom header to this distribution to only allow access to the s3 contents through cloudfront. Add a custom header called "Refere" and add a value to it that will be secret. This will ensure that s3 will only allow connections with the secret value in the "Referer" header.

Next, select "Redirect HTTP to HTTPS" because that is the goal here.

We also want to add the SSL cert that we just requested and validated. This cert must match the name exactly of the domain that covers the website. When we created the cert we used `example.com` and `*.example.com` so this will cover all references to the domain. If you messed this up, go back and just request a new cert, and delete the old one. (I had to do this!)

Lastly, I don't know if this is absolutely necessary, but just put in index.html (if that is your entry point) in the Default root object field.

Okay, that is ready to go.

### Fourth => Back to s3 permissions

Now that we have the header value in the distribution we need to include that in the bucket permissions. Head back to the bucket permissions, and add the "Condition" property with the secret value from the Custom Referer Header:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::BUCKETNAME/*",
            "Condition": {
                "StringLike": {
                    "aws:Referer": "somethingsecrethere"
                }
            }
        }
    ]
}
```

### Fifth => Make the DNS records
Lastly, in step five here, we want to make the DNS records that will point traffic to the cloudfront distribution which in turn forwards that traffic to the s3 bucket containing our static site content. 

The rub here is that the cloudfront name is, well, a name and not an ip address, so we can only make CNAME records for it. We will want to make a CNAME record for "www" with a value of the distribution name. 

But we also want to serve the apex (or naked) domain name, because we are trying to make it look like we know what we are doing. To do this, after much research, make an ALIAS record with the name field `@` and the value the same distrbution name.


After all of the above, we should have a static website served from s3 by way of cloudfront, which we SHOULD be able to reach from www or the apex `example.com`.

Note: the cloudfront distribution caches, and if you upload or change the site content files in the bucket, the old files will continue to be served. You can either invalidate the cache, or just make a change on the distribution which redeploys it. And the changes should now be seen.

### Conclusion

This very simple task took much longer than I anticipated, but when is it easy.

Enjoy!

![win](/assets/posts/s3-static-website/_gr-q9SaJezkOE-unsplash.jpeg)
Photo by <a href="https://unsplash.com/@grstocks?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">GR Stocks</a> 






*disclaimer: these musings are offered, at best, as educational, and at worst for entertainment purposes. Do not take action on the descriptions above, as they contain risks, and are not intended as financial advice. Do not do anything above.*