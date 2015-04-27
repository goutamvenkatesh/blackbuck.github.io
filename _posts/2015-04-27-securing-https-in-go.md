---
layout: post
title: "The easy guide to securing HTTP + TLS with Go"
modified:
categories: posts
excerpt: Go language conference is looking for good women speakers and Blackbuck Computing got selected to the NASSCOM Startup Warehouse!
tags: [GopherCon India, Blackbuck Computing, Fastah, Golang, Siddharth Mathur, Go programming language, Crypto, TLS, HTTPS, Excuse me your Crypto is showing]
#image: blackbuck-fastah-siddharth-mathur-gophercon-india.jpg
#  feature:
date: 2015-04-27T11:59:00+05:30
comments: false
share: true
---

The [Go programming language](http://golang.org) makes it easy to write and deploy servers offering HTTPS ( HTTP + Transport Layer Security) to clients. The crypto package in Go's standard library is easy to use and well documented: it's an under-explored gem. Due to it's low-on-legacy implementation of modern standards and easy configurability, there is no reason to insert an Apache or Nginx server to terminate TLS connections. A Go application server can do it all including being the front-end. All without OpenSSL!

Within the Go standard library, the http package provides the ```ListenAndServeTLS()``` API that offers secure HTTP to clients. Once a server is functionally working, how does one check HTTPS configuration and security? Here is a practical guide: 

1. Pick an audit tool to test your HTTP+TLS server
3. Understand any failures
4. Fix the issues
5. (Rinse and Repeat)

## PICK an audit tool
Let's start with the right tool. In this article, we use [scanner at Qualys SSL Labs](https://www.ssllabs.com/ssltest/). It's recommended by [Adam Langley](https://www.imperialviolet.org/2014/12/08/poodleagain.html), security maven and key author of Go's "crypto" codebase.
<figure>
	<img src="/images/SSL-labs-Start-Page.jpg">
</figure>	

## RUN the tool
  * Be prepared for failing grades, like the "C" below
  * Pay attention to the security warnings shown in color-coded message boxes. Work to fix them next. 
<figure>
	<img src="/images/SSL-Labs-C-grade.png">
</figure>

## UNDERSTAND and FIX the security issues
The first significant misconfiguration reported by the above scan is a POODLE attack vulnerability. 
While this is a bit of a false positive, you can make the tool pass this test by explicitly configuring the HTTPS server to advertise use of vTLS 1.0 or higher via ```tls.Config.MinVersion```:
{% raw %}
	// Make a server negotiate only TLS v1.0+ with clients
	config := &tls.Config{MinVersion: tls.VersionTLS10}
	s := &http.Server{TLSConfig: config}
	s.ListenAndServeTLS()
{% endraw %}

Next, is the yellow-coded warning about use of a weak cipher: RC4. You may fix this error by limited ciphers to a stronger set via ```tls.Config.CipherSuites```. Also notice the use of ```PreferServerCipherSuites``` member to force TLS negotiation to pick a result from the (presumably) stronger list of server-specified ciphers. 
{% raw %}
	// Work for iOS 6+ Safari, WP 8.x IE 11 and Android 4.4.2+
	config := &tls.Config{
		MinVersion:               tls.VersionTLS12,
		PreferServerCipherSuites: true,
		CipherSuites: []uint16{tls.TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,
			tls.TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,
			tls.TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA,
			tls.TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA,
			tls.TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA,
			tls.TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA,
			....
		},
	}
{% endraw %}

For a comprehensive understanding of the warnings flagged by the tool, check out [Mozilla's guide to TLS and cipher suites](https://wiki.mozilla.org/Security/Server_Side_TLS). 


## Rejoice at your passing grade!
Once you have fixed the issues flagged by the tool, an all-green report card awaits. 
<figure>
	<img src="/images/SSL-Labs-A-grade.png">
</figure>

A word of caution: Before changing Go's defaults, please do read the [standard library documentation TLS](http://golang.org/pkg/crypto/tls/), and [Mozilla's guide to TLS and cipher suites](https://wiki.mozilla.org/Security/Server_Side_TLS). Additional factors to consider while tuning the HTTPS server are:

 * network round-trips required to negotiate a TLS connection
 * encryption quality and CPU cost trade-off
 * legal implications if applicable. 

Happy security!

(_This topic was presented by the author at a lightening talk "Excuse me, your Crypto is showing!" during [GopherCon India 2015](http://www.gophercon.in)_)

<iframe src="https://www.slideshare.net/slideshow/embed_code/key/HQAB6OQ2mVoGDY" width="512" height="400" frameborder="5" marginwidth="0" marginheight="0" scrolling="no" sandbox="allow-scripts allow-same-origin allow-popups" seamless="seamless"></iframe>