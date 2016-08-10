haproxy
=========

Installs and configures HAProxy including SSL and SNI

Requirements
------------

None.

Role Variables
--------------

`_haproxy` contains the whole configuration. Please see `defailts/main.yml` for full configuration. Attention: This configuration needs to be adapted as there are a few place holders <...> for which no sensible default value may be used.

Dependencies
------------

None.

Example Playbook
----------------

```
    - hosts: servers
      vars:
        HAPROXY:
          use_default_config: False
          version: "1.5*"
          non_local_bind: False
          admin:
            ips:
              - "{{ ansible_default_ipv4.address }}:8404"
            user: admin
            password: "{{ S_HAPROXY.admin.password }}"
            uri: "/monitor"
          ciphers: "ALL:!aNULL:!eNULL:!EXPORT:!DES:!3DES:!MD5:!PSK:!LOW:!EXP:!DSS:!SEED:!ECDSA:!EDH:!CAMELLIA:!RC4:@STRENGTH"
          certs: "{{ S_HAPROXY.certs }}"
          extra: |+
            userlist fhemusers
            user fhem insecure-password {{ S_HAPROXY.fhem.password }}
          frontends:
            - name: heimbot.fritz.box
              ip: "{{ ansible_default_ipv4.address }}"
              tls: false
              port: 80
              backend:
              options:
                - forwardfor
                - httplog
              extra: |+
                acl is_my_domain hdr(host) -m end heimbot.fritz.box
                redirect location https://heimbot.fritz.box code 301 if !is_my_domain
                redirect scheme https code 301 if !{ ssl_fc }
            - name: tls-sni
              ip: "{{ ansible_default_ipv4.address }}"
              tls: true
              port: 443
              cert_id: heimbot_fritz_box
              backend: heimbot.fritz.box
              options:
                - forwardfor
                - httplog
              sni: []
              extra:
            - name: Fhem
              ip: "{{ ansible_default_ipv4.address }}"
              tls: yes
              port: 8083
              cert_id: heimbot_fritz_box
              backend: Fhem
              options:
                - forwardfor
                - httplog
              extra: |+
                # basic auth is deactivated # acl auth_acl http_auth(fhemusers)
                # basic auth is deactivated # http-request auth realm basicauth unless auth_acl
          backends:
            - name: heimbot.fritz.box
              balance: leastconn
              cookie: CD insert preserve
              options:
                - httplog
                - httpchk
              defaults:
                port: 80
                option: check cookie localhost
              servers:
                - localhost
            - name: Fhem
              balance: leastconn
              options:
                - httplog
                - httpchk
              defaults:
                port: 8083
                option: check cookie localhost
              servers:
                - localhost
          error_pages:
            title: Heimbot
            e_mail: info@{{ ansible_fqdn }}
            phone: +91 111 0
            pages:
              - { code: 400, description: "Bad request", comment: "Ihr Browser hat eine ung&uuml;ltige Anfrage gesendet." }
              - { code: 401, description: "Unauthorized", comment: "Die Anfrage kann nicht ohne gültige Authentifizierung durchgeführt werden." }
              - { code: 403, description: "Forbidden", comment: "Die Anfrage Ihres Browsers ist verboten." }
              - { code: 408, description: "Request Time-out", comment: "Die Anfrage Ihres Browsers hat zu lange gedauert." }
              - { code: 500, description: "Server Error", comment: "Ein interner Server Fehler ist aufgetreten." }
              - { code: 502, description: "Bad Gateway", comment: "Der Server hat eine ung&uuml;ltige oder unvollst&auml;ndige Antwort geschickt." }
              - { code: 503, description: "Service Unavailable", comment: "Zur Zeit ist kein Server zur Beantwortung Ihrer Anfrage verf&uuml;gbar." }
              - { code: 504, description: "Gateway Time-out", comment: "Die Antwort des Servers hat zu lange gedauert." }
        roles:
        - { role: haproxy, tags: ['haproxy'], _haproxy: "{{ HAPROXY }}" }
```

License
-------

See LICENSE file.

Author Information
------------------

Initially created by Lukas Pustina [@drivebytesting](https://twitter.com/drivebytesting).

