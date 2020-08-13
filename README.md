blauwe_lucht.docker_webapps
=========

Ansible role to run multiple webapps on https on a single Docker host,
signed with self-signed or Let's Encrypt certificates.

This role creates a reverse proxy container running nginx and configures nginx
to forward requests to the registered webapps. If requested each webapp will
request a Let's Encrypt certificate so the webapps can be reached with HTTPS.
These certificates will be automatically renewed through a certbot container.
If Let's Encrypt is not used self-signed certificates for HTTPS will be generated.

Current state
-------------

Alpha. Only use this role to test it. No molecule tests have been written yet.

Requirements
------------

The node should already be running Docker. This can for example be done with
the role [geerlingguy.docker](https://galaxy.ansible.com/geerlingguy/docker).

Role Variables
--------------

#### docker_webapps
Configuration of the webapps to run. Example:

```yaml
docker_webapps:
  - name: samtris
    docker_image: blauwelucht/samtris:v2.0
    port: 8080
    volumes:
      - /var/log:/var/log
```

###### name
Both the name of the Docker Compose service that will be generated
and the name of the subdomain that the reverse proxy will listen to.

###### docker_image
The image that will be used to host the webapp.
It's recommended to use a tag since this role will not check for updated
Docker images.

###### port
The port on the container where the webapp is listening.

###### volumes
Volumes are optional, they will be copied verbatim into the generated
docker-compose.yml.

#### docker_webapps_use_lets_encrypt
Set use_lets_encrypt only to true when the Docker host can be reached from
the internet. This is needed by Let's Encrypt to verify a certificate request.
When use_lets_encrypt is false, a self-signed certificate will be used.

#### docker_webapps_domain_name
The domain name where all the subdomains are part of.

#### docker_webapps_email_address
#### docker_webapps_organization_name
#### docker_webapps_country_name
The email address,  organization and country used to request both
self-signed and Let's Encrypt certificates.

#### docker_webapps_docker_compose_project
The project name for Docker Compose. Only needs to be overwritten when there's
already a Docker Compose project with the same name.

#### docker_webapps_reverse_proxy_container_name
The name of the reverse proxy container. Only needs to be overwritten when
there's already a container with that name.

#### docker_webapps_certbot_renew_container_name
The name of the certbot container for certificate renewal. Only needs to be
overwritten when there's already a container with that name.

#### docker_webapps_certbot_staging_param
When playing around, set docker_webapps_certbot_staging_param to
"--staging" so you won't hit the Let's Encrypt rate limits.

Dependencies
------------

None

Example Playbook
----------------

```yaml
- hosts: servers
  roles:
  - name: blauwe_lucht/docker_webapps
    vars:
      docker_webapps:
      - name: samtris
        docker_image: blauwelucht/samtris:v2.0
        port: 8080
      docker_webapps_use_lets_encrypt: true
      docker_webapps_domain_name: example.com
      docker_webapps_email_address: someone@example.com
      docker_webapps_organization_name: ACME
      docker_webapps_country_name: NL
```

Notes
-----

- Make sure the FQDN ```<site.domain>``` is resolved to your server.
The FQDN is used by the reverse proxy to route requests to the right webapp.
- For each webapp nginx listens to an extra site name is registered to make it possible to test your Ansible
scripts on a test server: ```<site>-tst.<domain>```. 
- The best way to handle updates to the webapp containers is to use tags in the
image names. When updating the tag in the docker_webapps configuration variable,
the change will be detected and the new images will be pulled and used to replace
the existing container.
- To delete/refresh a Let's Encrypt certificate execute ```certbot delete --cert-name <fqdn>```
in the certbot container, otherwise some history will stick around and you will get new certificates with a
-0001 prefix which will not be found by nginx.
_IMPORTANT_: only delete/refresh one certificate at a time, otherwise nginx will fail to start because one of the sites
has a missing SSL certificate while the conf still specifies one. (I haven't figured out a way to fix this yet).
- When not using Let's Encrypt certificates the webapps can still be accessed
over HTTPS, but you will get a warning in your browser that the certificate
is self-signed. For those webapps you can safely ignore those warning.

License
-------

BSD

Author Information
------------------

This role was created in 2020 by Blauwe Lucht.
