# DHANE
Dehydrated Authentication of Named Entities

A plug-in for [dehydrated](https://github.com/dehydrated-io/dehydrated) to create DNS TLSA records.

## Branch: powerdns-api-migration

### Overview
This branch aims to modernize the DNS management functionality in DHANE by replacing the current `pdnsutil` command-line tool usage with direct PowerDNS HTTP API calls.

### Current Status
🚧 **In Development** - Initial planning and development phase

Track: [Move from `pdnsutil` to PowerDNS API calls #2](https://github.com/alainwolf/DHANE/issues/2)

### Goals
- PowerDNS HTTP API requests instead of the `pdnsutil` command-line utility;
- Move configuration variables to the dehydrated configuration file
  - inlcudes possiblity to set certificate specific configuratoion options
- Maintain compatiblity and re-use configuration values from PDNS hook script

### Changes Planned

#### DNS Functions to Migrate
The following functions in [`dehydrated_tlsa`](dehydrated_tlsa) will be updated:

- `create_dns_rr()` - Replace `pdnsutil add-record` with API POST requests
- `delete_dns_rr_set()` - Replace `pdnsutil delete-rrset` with API DELETE requests
- Zone management operations:
  - Replace `pdnsutil rectify-zone` with API rectify endpoint
  - Replace `pdnsutil increase-serial` with API serial increment
  - Replace `pdnsutil check-zone` with API validation

#### New Configuration Options
Additional settings will be added to support API connectivity:
- PowerDNS API endpoint URL
- API authentication key
- HTTP timeout settings
- SSL/TLS verification options

### Benefits
- **Better Integration**: Direct API calls provide more reliable communication
- **Enhanced Error Handling**: Structured JSON responses vs. parsing command output
- **Improved Performance**: Eliminate subprocess overhead from `pdnsutil` calls
- **Modern Architecture**: Move away from command-line dependency to HTTP API
- **Better Monitoring**: Structured logging and response validation

### Compatibility
The branch will maintain full compatibility with existing [`tlsa.txt`](README.md#defining-tlsa-records) configuration files and dehydrated hook integration.

### Requirements

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
