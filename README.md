# DHANE
Dehydrated Authentication of Named Entities

A plug-in for [dehydrated](https://github.com/lukas2511/dehydrated) to create DNS TLSA records.

## Features

 * Automated creation of TLSA entries in DNS with every new certificate.
 * Supports dehydrated roll-over keys.

## Requirements

### [dehydrated](https://github.com/lukas2511/dehydrated)

Obviously.


### [lexicon](https://github.com/AnalogJ/lexicon)

Manipulate DNS records on various DNS providers in a standardized way.

Lexicon provides a way to manipulate DNS records on multiple DNS providers in a
standardized way. Lexicon was designed to be used in automation, specifically 
[Let's Encrypt](https://letsencrypt.org/).


### Others

 * This script uses the DNS lookup utility **dig** for all DNS queries.
 * Like dehydrated, **OpenSSL** is used for certificate and key operations and hashing.


## Installation


### dehydrated

I assume you are already familiar with dehydrated and use it to get your Let's
Encrypt certificates.


### lexicon

lexicon is available as a Python Installation Package (PIP):

See the [lexicon setup instructions](https://github.com/AnalogJ/lexicon#setup).


## Configuration

### Hook-up with dehydrated

Call it in your dehydrated 
[hook script](https://github.com/lukas2511/dehydrated/blob/master/docs/examples/hook.sh) 
with the **deploy_cert()** and maybe also **unchanged_cert()** hook.

```
deploy_cert() {
    local DOMAIN="${1}" KEYFILE="${2}" CERTFILE="${3}" FULLCHAINFILE="${4}" CHAINFILE="${5}"

    # This hook is called once for each certificate that has been
    # produced. Here you might, for instance, copy your new certificates
    # to service-specific locations and reload the service.
    # Create TLSA records in DNS.

./dehydrated_tlsa "deploy_cert" "${DOMAIN}" "${KEYFILE}" "${CERTFILE}" "${FULLCHAINFILE}" "${CHAINFILE}"

```


### Defining TLSA Records

For certificates which need TLSA DNS records, create a file called **tlsa.txt** in the certificates directory and add the FQDN records one by line:

```
#/etc/dehydrated/certs/mail.example.net/tlsa.txt
#
# DANE TLSA records for the mail.example.net certificate
#
_25._tcp.mail.example.net
_443._tcp.webmail.example.net
_443._tcp.webmail.example.org

```
Empty lines, spaces and lines beginning with # are ignored.

The DNS records can be in different domains, as long as **pdnsutil** is able to create records in it.


### Configuration

You need to set the IP address of your DNS master in the script. 

Besides that you can customize the TTL of your DNS records, and pick your favorites from the usual menu of TLSA **usage flags**, **selectors** and **mtypes**, While the set defaults are sane, selector 0 (full certificate in DNS) is not implemented.


## Limitations

 * It never deletes any DNS records. Just checks if the required ones are already there and adds new ones as needed.
 * It won't create TLSA records of un-hashed full certificates or public keys (mtype 0). 
 * While checking existing DNS records, it just compares the record names and SHA2 hash values. 
   It doesn't look at the usage flags,  selectors and mtypes. I never had a use case where this should be necessary.
 * Did I mention that this hasn't been tested much?

## Warnings
This has been cobbled together rather quickly and it hasn't been tested very much.

Its easy to shoot yourself in the foot with TLSA entries, and render your **services unusable** and **all of your mail undeliverable**. 

After that has happened, it will take at minimum the TTL of your DNS records, to pull the bullet out of your foot again and stop the bleeding.

Please try this at home, with some inexistent or unimportant services, first.

## Thanks

 * To [lukas2511](https://github.com/lukas2511) and contributors for dehydrated. 
 * To [Let's Encrypt](https://letsencrypt.org/) for the tremendous help encrypting the universe planet by planet.
