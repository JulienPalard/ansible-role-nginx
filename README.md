# Nginx host

This role sets up nginx with HTTPS using letsencrypt.

It tries to be minimal.


## Role Variables

- `nginx_domain`: Domain name of your website, like:
  `example.com`. Also used as a name for the configuration file.
- `nginx_certificates (default: `[]`)`: A list of HTTPS
  certificates to generate, like `[www.example.com]`. The first one is
  used as the certificate name and nginx snippet file name. Giving
  multiple domains creates a single certificate, and a single snippet,
  for all of them.
- `nginx_owner` (optional): Create unix system account for this website, like `example_com`.
- `nginx_path` (optional): Path for your static files, like: `/var/www/example.com/`.
- `nginx_public_deploy_key` (optional): A public SSH key, so you can push your static files via rsync.
- `nginx_conf`: The whole nginx configuration file, the default works for a static website.


## Example Playbook

This will create a user, a directory, configure (see
`defaults/main.yml`) and generate an HTTPS certificate.

```yaml
- hosts: static_websites
  tasks:
    - name: Setup mdk.fr
      include_role: name=julienpalard.nginx
      vars:
        nginx_domain: mdk.fr
        nginx_owner: mdk_fr
        nginx_path: /var/www/mdk.fr/
        nginx_certificates: [mdk.fr, www.mdk.fr]

```

Or personalize the configuration as needed:

```yaml
- name: Setup palard.fr
  include_role: name=julienpalard.nginx
  vars:
    nginx_domain: palard.fr
    nginx_certificates: [palard.fr, julien.palard.fr, www.palard.fr]
    nginx_conf: |
      server
      {
          listen 80;
          server_name .palard.fr;
          access_log /var/log/nginx/palard.fr-access.log;
          error_log /var/log/nginx/palard.fr-error.log;
          return 301 https://$host$request_uri;
      }

      server
      {
          listen 443 ssl;
          server_name .palard.fr;
          access_log /var/log/nginx/palard.fr-access.log;
          error_log /var/log/nginx/palard.fr-error.log;
          root /var/www/palard.fr/;
          include snippets/letsencrypt-palard.fr.conf;  # Notice the name of the snippet, generated from the first certificate name.
          index index.html;
          location / {return 301 https://mdk.fr;}
      }
```

Beware of using `include_role` and not `roles` so variables from a
website does not *leak* to others. It would not be nice if a random
`nginx_public_deploy_key` is given to all your site not having one.


### License

MIT


### Author Information

Julien Palard — https://mdk.fr
