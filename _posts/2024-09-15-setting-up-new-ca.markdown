---
layout: post
title:  "Setting up own Certificate Authority (CA)"
date:   2024-09-15 18:25:00 +0200
categories: how-to pihole certificates
tags: ["raspberry pi", "how to", "pihole", "certificates"]
---

# Setting up our own Certificate Authority (CA)

**I'm back!!!** These past 8 months have been intense and I could consider myself lucky whenever I found some time to rewind - while studying, preparing exams and getting my final project done. Writing this blog or getting any other IT project done definitely wasn't on the plan.

After all this time I finished my studies and now I'm a certified IT professional :) it has been a wild trip. Now I can say it has been worth every minute and I'm glad I took the risk of changing my professional career at 30+... but honestly I wouldn't do it again and would encourage everyone to think twice before taking such a decision.

But now all of that is past behind me and I can concentrate on other projects and improve my IT skills on the way.

As I didn't have much time so start any new project, I decided to improve those I had already started. The web UI of my Pihole was SSL-ready, but no device would trust my self-signed certificate. 
What about creating my own CA and make my home devices trust its root certificate? This way, all certificates signed by my CA would be immediately trusted. Later on, I could even issue user certificates to grant access into my network. Many possibilities ahead.

You can find a ton of tutorials to make this possible - I just wrote down here what I did.


## Disclaimer & Credits

> Brad Touesnard's article [Installing your Root Certificate](https://deliciousbrains.com/ssl-certificate-authority-for-local-https-development/#installing-root-cert) helped me a lot getting this done.
>
> Note that after following this tutorial, your own issued certificates **won't be 100% trusted** by all computers and browsers, since our CA isn't an official one. Nevertheless it's a valid way of having trusted certificates to encrypt dev sites and Web UIs for our IT projects.


## Generate CA's private key and certificate

* [Optional] Create store location for certificates: ```mkdir ~/certs```
* Generate private key for the CA: ```openssl genrsa -aes256 -out <name-of-private-key-file> 4096```
* When prompted, set a passphrase for the private key - recommended for protecting the file.
* Generate a root certificate and sign it with the private key: ```openssl req -x509 -new -nodes -key <name-of-private-key-file> -sha256 -days 1095 -out <name-of-certificate>.pem``` . You will be prompted for additional information when generating the certificate - you can press Enter to leave the info fields blank. Alternatively, you can pass this information as a command line argument - take a look below at the ```'-subj'``` parameter.

## Distributing the certificate

Unless you distribute your root certificate to other computers, those won't trust any certificate coming from your own CA. You can distribute this root certificate to devices in your local network, so that their browsers will accept self-signed certificates as trustworthy - those self-signed certificates can be used to encrypt HTTP own dev sites and Web UIs (such as the Pi Hole Admin Console).

* Instructions for **Windows, MacOS & iOS**: check out Brad Touesnard's article, under [Installing your Root Certificate](https://deliciousbrains.com/ssl-certificate-authority-for-local-https-development/#installing-root-cert)
* Instructions for **Linux**: copy the .pem certificate file into /usr/local/share/ca-certificates - ```sudo cp ~/certs/<name-of-certificate>.pem /usr/local/share/ca-certificates```
* Instructions for **Android**: check out Google's official docs (works also for devices other than Pixel) - [Add & remove certificates](https://support.google.com/pixelphone/answer/2844832?hl=en)

## Issuing certificates with the CA

These certificates will be signed with the private key we created in the first place. They are still 'self-signed' in a way, since our CA is not officially recognized. But as we have saved our CA's root certificate in other devices, these will accept other certificates signed by our CA.

* Generate private key: ```openssl genrsa -out <name-of-private-key-file> 4096```
* Create a Certificate Signing Request (CSR): ```openssl req -new -key <name-of-private-key-file> -out <name-of-csr>.csr```
* Create an extension config file, which will define **Subject Alternative Names** for the site - DNS names and IP addresses that will be associated with this particular certificate: ```touch <name-of-extfile>.ext``` and paste the following contents

```console
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = hostname.local
IP.1 = <IP-Address>
```


* Use the CSR together with the config file to issue a certificate and sign it with our CA's private key: ```openssl x509 -req -in <name-of-csr>.csr -CA <name-of-CA-certificate>.pem -CAkey <name-of-CA-private-key>.key -CAcreateserial -out <name-of-certificate>.crt -days 1085 -sha256 -extfile <name-of-extfile>.ext```
* If you intend to use the certificate for a web service like **lighttpd**, it might require a conversion into .pem - concatenate the private key and the .cert-file into a .pem-file for this: ```cat <name-of-certificate>.crt <name-of-private-key-file>.key <name-of-chain>.pem```


### Breaking out the commands

```console
openssl genrsa -aes256 -out <name-of-private-key-file> 4096
```

* ```genrsa```: generate a RSA private key
* ```-aes256```: cipher which will be used to encrypt the key. AES provides the highest level of security.
* ```-out <input>```: file where to output the private key
* ```4096```: length of the key in bits - the higher, the better. It defaults to 2048.

```console
openssl req -x509 -nodes -key <name-of-private-key-file> -sha256 -days 1095 -out <name-of-certificate>.pem
```

* ```req```: request and generate a PKCS#10 certificate
* ```-x509```: outputs a certificate, instead of a certificate request. 
* ```-new```: use a new certificate request (default value for -x509). Alternatively, you can specify a certificate request with ```-in``` (input file)
* ```-nodes```: deprecated since OpenSSL 3.0, you can use ```-noenc``` instead. Specifies that, if a private key is created, it won't be encrypted.
* ```-key <input>```: name of the private key which will be used to sign the certificate
* ```-sha256```: hashing algorithm
* ```-days <amount>```: validity of the certificate in days (should be therefore a multiple of 365)
* ```-out <output>```: output file to save the certificate (use .pem format)
* ```-subj 'Info'```: you can pass the information as an argument instead of on prompt. Use for this the format specified in the [OpenSSL official documentation](https://docs.openssl.org/3.0/man1/openssl-req/#options)


## Last tweak - from CRT to PEM

* Depending on the web engine we're using, it might accept .pem-files instead of .crt-certificates only - I had this issue using **lighttpd on PiHole**.
* A workaround would be redirecting the output of the certificate file (.crt) and the private key (.key) files into a .pem file: ```cat <name-of-certificate>.crt <name-of-private-key-file>.key > <name-of-pem-file>.pem```
* If redirecting the output doesn't work and you need to copy-paste the contents, be careful not to add any newline breaks without noticing.
* As the resulting .pem-file contains also the private key, be sure to adjust the file permissions accordingly - ```chmod 600 <name-of-pem-file>.pem```


## Final Results

* When browsing our site for the first time, the browser will warn us about an **unknown signing CA**. Once we accept the risk and continue, when browsing the site again the certificate will be shown as trusted. However, we will still see a warning that the **Certificate Authority isn't publicly known**.
* For local and testing purposes these certificates are more than acceptable I think - for next projects I'd like to investigate if I can sign device certificates for WLAN authentication.
