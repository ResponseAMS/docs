# Requirements

## Hosting Environment

Response is intended to be deployed in our cloud hosting environment (for a low monthly cost), or using your own web
server. At the current time, we do not support installing Response on shared hosting. If you'd like to help us test
Response on shared hosting, we'd be willing to work with you.

If you are comforatble running your own web server, a service like Vultr, Linode, DigitalOcean, and others would be
sufficient.

## System Requirements

Response has a few system requirements that are easily met in local developemnt environments as well as web server
environments.

The following table shows the minimum hardware/virtual hardware requirements for running Response based on the number
of users you expect to have using Response at the same time.

<!-- title: Server Requirements -->

| Simultaneous Active Users | Memory |
| ------------------------- | ------ |
| 1-40                      | 2 GB   |
| 41-100                    | 3 GB   |
| 100+                      | 4+ GB  |

Remember that Response will only perform as well as its database. You can increase your memory as needed but the above
should be a good starting point. It is also strongly recommended that you configure Response to use the Redis caching
implementation which prevents excessive database calls for the same data.

## Web Server Requirements

- PHP >= 7.2
- PHP Extensions:
  - BCMath
  - Ctype
  - JSON
  - Mbstring
  - OpenSSL
  - PDO
  - Tokenizer
  - XML
  - phpredis (optional but recommended)
