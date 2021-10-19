# BOSH-deployed nginx Server

This [BOSH](https://bosh.io/) release deploys an nginx webserver.

***Warning: You may receive HTTP 403 Status ("forbidden") or see  "permission
denied" errors in your logs when using stemcells >= 3541.x; to fix, set the
worker's UNIX group to `vcap` at the top of your `nginx_conf` property with the
following line:***

```
user nobody vcap; # group vcap can read most directories
```

### 0. Quick Start

#### 0.0 Quick Start: Pre-requisites

You must have a BOSH Director and have uploaded stemcells to it. Our examples assume the [BOSH CLI v2](https://github.com/cloudfoundry/bosh-cli).

Follow the instructions to install BOSH Lite: <https://bosh.io/docs/bosh-lite>;
upload the Cloud Config, set the routes, but no need to deploy Zookeeper.

Upload Ubuntu stemcell

```bash
bosh -e vbox us https://s3.amazonaws.com/bosh-core-stemcells/warden/bosh-stemcell-3468-warden-boshlite-ubuntu-trusty-go_agent.tgz
```

Clone the nginx repository:

```bash
cd ~/workspace
git clone https://github.com/cloudfoundry-community/nginx-release.git
cd nginx-release
```

#### 0.1 Quick Start: Upload release to BOSH Director

```bash
bosh -e vbox ur https://github.com/cloudfoundry-community/nginx-release/releases/download/1.21.3/nginx-release-1.21.3.tgz
```

#### 0.2 Quick Start: deploy

(This assumes you're in the `~/workspace/nginx` directory cloned in a previous step):

```bash
bosh -e vbox -d nginx deploy manifests/nginx-lite.yml
```

#### 0.3 Quick Start: test

Browse to <http://10.244.0.34/>; you should see the following:

![nginx_release_welcome](https://user-images.githubusercontent.com/1020675/27837760-14599acc-609b-11e7-8e1a-eb4d305be2b7.png)

### 1. Post-deployment HTML content

We find it effective to set the
[`pre_start`]((https://bosh.io/docs/pre-start.html)) property to populate the
webserver content. See
[here](https://github.com/cunnie/deployments/blob/d47af699bf11c4b168abfb9d5119ecc6dfddc06f/etc/nginx.yml#L53-L67)
for an example (sslip.io).

Alternatively, you may manually add the HTML content *after* successful deployment.

We recommend installing HTML content on the persistent disk, e.g.
`/var/vcap/store/nginx/document_root/` so that subsequent redeploys
do not require re-installation of HTML content, i.e. the
`nginx.conf` should have the following directive:

```
server {
  root /var/vcap/store/nginx/www/document_root;
```

## Notes

#### 1. nginx Job Properties

* `nginx_conf`: *Required*. This contains the contents of nginx's configuration
  file, nginx.conf. Here is the beginning from a sample configuration:
  ```yaml
    nginx_conf: |
      worker_processes  1;
      error_log /var/vcap/sys/log/nginx/error.log   info;
  ```

* `ssl_key`: *Optional*, defaults to ''. This contains the contents of the
  SSL key in PEM-encoded format. This is required if deploying an HTTPS webserver.
  The key is deployed to the path `/var/vcap/jobs/nginx/etc/ssl.key.pem` and
  requires the following line in the `nginx_conf`'s *server* definition:

  ```
  ssl_certificate_key /var/vcap/jobs/nginx/etc/ssl.key.pem;
  ```

  Here is the beginning from a sample configuration:

  ```yaml
  ssl_key: |
    -----BEGIN RSA PRIVATE KEY-----
    MIIJKQIBAAKCAgEAyv4in6scMw3OkBlr++1OooLuZQKftmwGIO8puOj6lSH4H1LI
  ```

* `ssl_chained_cert`: *Optional*, defaults to ''. This contains the contents of the
  SSL certificate in PEM-encoded format. This file will most likely contain
  several chained certificates.  The certificate for the server should appear
  at the top, followed by the intermediate certificate.  This property is
  required if deploying an HTTPS webserver.  The certificate is deployed to the
  path `/var/vcap/jobs/nginx/etc/ssl_chained.crt.pem` and requires the
  following line in the `nginx_conf` *server* definition:

  ```
  ssl_certificate     /var/vcap/jobs/nginx/etc/ssl_chained.crt.pem;
  ```

  Here is the beginning from a sample configuration:

  ```yaml
  ssl_chained_cert: |
    -----BEGIN CERTIFICATE-----
    MIIGSjCCBTKgAwIBAgIRAOxg+vyhygau6bc2SAooL6owDQYJKoZIhvcNAQELBQAw
  ```

* `pre_start`: *Optional*, contains a pre-start script to execute,
useful for populating web content.

## Developer Notes

Developer notes, such as building and test a release, are available [here](docs/DEVELOPER.md).
