# Postal Config Outline

The settings for my production Postal deployment. This is only intended to act as a guide/explanation for settings. You'll want to make the changes in the actual `postal.yml` configuration file that's in the directory with this.

## Web Server Settings.

Change the `host` value to the hostname you want to use for the web service.


```yaml
web:
  host: postal.example.com
  protocol: http
```

----

## General Settings.

I' not sure if these settings apply to every Postal application container, or if it's just the Web server.

I'm not using `ip pools` so I've left it at false.

I'm going to toy with the `use_local_ns_for_domains` setting to see what it does. I've left everything else alone.

```yaml
general:
  use_ip_pools: false
  exception_url:
  maximum_delivery_attempts: 18
  maximum_hold_expiry_days: 7
  suppression_list_removal_delay: 30
  use_local_ns_for_domains: false
  default_spam_threshold: 5.0
  default_spam_failure_threshold: 20.0
  use_resent_sender_header: true
  ```

----

## Web Server

You can leave this alone unless you want the web server to listen on a specific IP address.

```yaml
web_server:
  bind_address: 0.0.0.0
  port: 5000
  max_threads: 5
  ```

----

## Main DB 

As the name says, this applies to the SQL server. Specifically the DB used for storing your main postal settings, users, and reporting data.

Enter the IP address of your SQL server, as well as the password along with the DB and Username if you've changed them, I recommend that you leave the username and database as `postal`

```yaml
main_db:
  host: <IP-address-of-db-server>
  port: 3306
  username: postal
  password:
  database: postal
  pool_size: 5
  ```

----

## Logging

Here we find the logging settings.

The only change I've made is I've enabled Graylog logging.

```yaml
logging:
  stdout: false
  root: # Automatically determined based on config root
  max_log_file_size: 20
  max_log_files: 10
  graylog:
    host: <IP-address-of-graylog-server>
    port: 12201
```

----

## message_db

This is the SQL db used for your messages.

I also recommend that you keep the user and db name the same.

```yaml
message_db:
  host: <IP-address-of-db-server>
  port: 3306
  username: postal
  password:
  prefix: postal
```

----

## RabbitMQ

You only need to worry about the `host` and `password` fields for this section, I don't recommend you enable TLS unless it's required in your environment.

```yaml
rabbitmq:
  host: <IP-address-of-rmq-server>
  port: 5672
  tls: false
  verify_peer: true
  tls_ca_certificates:
    - /etc/ssl/certs/ca-certificates.crt
  username: postal
  password:
  vhost: /postal
  ```

----

## Workers

I haven't bothered to change these settings, you can tweak them if you run in to performance issues.

```yaml
workers:
  quantity: 1
  threads: 4
```

----

## SMTP Server

I haven't changed these settings. We terminate TLS on our load balancers to simplify certificate management. 14MB is far more than enough attachments that I'll be sending.

```yaml
smtp_server:
  port: 25
  bind_address: '::'
  # to enable SMTPS we'll need to generate a certificate pair, we can use an acme script or
  tls_enabled: false
  tls_certificate_path: # Defaults to config/smtp.cert
  tls_private_key_path: # Defaults to config/smtp.key
  tls_ciphers:
  ssl_version: SSLv23
  proxy_protocol: false
  log_connect: true
  strip_received_headers: false
  max_message_size: 14 # size in Megabytes
```

----

## SMTP Relays

This is useful if you want postal to utilize an SMTP Relay to send mail. You can't do any mail routing though so if you define a relay here, all of the mail will be sent to it.

```yaml
smtp_relays:
  -
    hostname:
    port: 25
    ssl_mode: Auto
```

----

## DNS Records

This is the section where you define the DNS records that Postal uses and will expect to be able to resolve. This specifically only applies to the main domain that you're using for the Postal server, you can still add as many other domains as you'd like, but you'll still add the SPF, MX, DKIM, etc records to those domains.

I've set them all to my domain and have changed the `dkim_identifier` from postal to something else like `dkim_relay` in an effort to obfuscate what SMTP server I'm using for the same reason that the Postal team has put in the comments.

```yaml
dns:
  mx_records:
    - mx.mail.example.com
  smtp_server_hostname: smtp.example.com
  spf_include: spf.mail.example.com
  return_path: rp.mail.example.com
  route_domain: routes.mail.example.com
  track_domain: track.mail.example.com
  helo_hostname: # By default, this will be the same as the `smtp_server_hostname`
  dkim_identifier: postal # You're welcome to change this value to whatever you'd like, I don't like to leak internal software publicly if I can avoid it.
  domain_verify_prefix: mail-verification
  custom_return_path_prefix: mlrp
```

## Internal SMTP

This section is for configuring the Postal app itself and the SMTP server that you want it to use for things like web user password resets.

By default they have you using the Postal SMTP server itself for that function, but if you want to use a separate server for than then you can change the `host` value as well as provide a username/password if you need to.

```yaml
smtp:
  host: 127.0.0.1 # Either localhost, or whatever SMTP host you'd like to use, this is for the web server so in this situation it would be something like the following. host:<smtp_server_hostname>
  port: 25
  username: # Complete when Postal is running and you can
  password: # generate the credentials within the interface.
  from_name: Postal
  from_address: postal@yourdomain.com
  ```

----

## Misc Settings

The remaining are miscellaneous settings that I've more or less ignored. 

The rails secret key is something that's generated by the app, so leave that alone.

I don't have clamav, rspamd, or spamd enabled for my environment as this is exclusively an outbound mail relay, and I really don't feel like fighting internal teams to tweak/tune any incidental outbound spam filtering that may take place.

```yaml
rails:
  environment: production
  secret_key:

rspamd:
  enabled: false
  host: 127.0.0.1
  port: 11334
  ssl: false
  password: null
  flags: null

spamd:
  enabled: false
  host: 127.0.0.1
  port: 783

clamav:
  enabled: false
  host: 127.0.0.1
  port: 2000

smtp_client:
  open_timeout: 30
  read_timeout: 60
```