# DHANE
Dehydrated Authentication of Named Entities

A plug-in for [dehydrated](https://github.com/dehydrated-io/dehydrated) to create DNS TLSA records.

## Requirements

### PowerDNS Authoritative Server
Since I use PowerDNS myself, its written for its command and control utility.
The script calls **pdnsutil** to create the records in DNS.
It also uses **dig** for DNS queries.

## Installation
Call it in your dehydrated [hook script](https://github.com/dehydrated-io/dehydrated/blob/master/docs/examples/hook.sh) with the **deploy_cert()** and maybe also **unchanged_cert()** hook.

```
deploy_cert() {
    local DOMAIN="${1}" KEYFILE="${2}" CERTFILE="${3}" FULLCHAINFILE="${4}" CHAINFILE="${5}" TIMESTAMP="${6}"

    # This hook is called once for each certificate that has been
    # produced. Here you might, for instance, copy your new certificates
    # to service-specific locations and reload the service.
    # Create TLSA records in DNS.

./dehydrated_tlsa "deploy_cert" "${DOMAIN}" "${KEYFILE}" "${CERTFILE}" "${FULLCHAINFILE}" "${CHAINFILE}" "${TIMESTAMP}"

```

## Defining TLSA Records

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

## Configuration

You need to set the IP address of your DNS master in the script.

Besides that you can customize the TTL of your DNS records, and pick your favorites from the usual menu of TLSA **usage flags**, **selectors** and **mtypes**, While the set defaults are sane, selector 0 (full certificate in DNS) is not implemented.

## Limitations

 * It never deletes any DNS records. Just checks if the required ones are already there and adds new ones as needed.
 * It won't create TLSA records of un-hashed full certificates or public keys (mtype 0).
 * While checking existing DNS records, it just compares the record names and SHA2 hash values. It doesn't look at the usage flags, selectors and mtypes. I never had a use case where this should be necessary.

## ACKs

Most thanks go to [lukas2511](https://github.com/lukas2511) and contributors for dehydrated. And to Let's Encrypt of course for encrypting the whole universe planet by planet.
