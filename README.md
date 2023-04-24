# Wir bauen uns eine Labor-Umgebung

Ziel:

- installieren aller Dienste mittels docker compose
- problemloser Zugriff ohne weitere lokale Konfigurationen

## Quay - eine eigene Docker registry

Das upstream opensource Projekt für die Red Hat Quay Registry baut und veröffentlicht die Quay Registry als Docker Image auf <https://quay.io/repository/project-quay/quay>. Davon benutzen wir das image `quay.io/projectquay/quay:latest`.

Wir werden Quay einsetzen als Docker Registry.

## Sonatype Nexus OSS

Sonatype veröffentlicht ihren Nexus Repository Manager als Docker Image auf <https://hub.docker.com/r/sonatype/nexus3>. Davon benutzen wir das image `sonatype/nexus3:latest`.

Wir werden Nexus einsetzen als Docker Registry, als Maven Repository und als NPM Repository.

## Traefik als Reverse Proxy mit SSL Terminierung

Traefik ist ein Reverse Proxy mit SSL Terminierung. Wir benutzen das Image `avogt/traefik-with-localhost-tls:2.10`. Dieses habe ich selber von traefik:2.10 abgeleitet, ergänzt um ein SSL-Zertifikat für `*.localhost.direct` -  ein sogenanntes wildcard-Zertifikat. Dieses Zertifikat wird von Traefik verwendet, um SSL-Verschlüsselung für hostnames in der domain `localhost.direct` so anzubieten, dass alle Browser und http-basierte Werkzeuge wie `maven`, `npm`, `docker`, `kubernetes` usw. damit zurecht kommen. Dieses Image ist auf <https://hub.docker.com/r/avogt/traefik-with-localhost-tls> zu finden, der source code findet sich unter <https://github.com/avogt-sundn/traefik-with-localhost-tls>.

Das benutzte SSL-Zertifikat ist nicht auf eine IP-Adresse beschränkt. Das erlaubt uns, den traefik als Webserver auf einer von uns frei gewählte IP-Adresse erreichbar zu machen. Es fehlt nur der DNS-Eintrag, der die IP-Adresse mit dem Domainnamen verknüpft. Diesen Eintrag können wir selbst vornehmen, lokale in der `/etc/hosts` Datei oder mit Hilfe eines DNS service, der unter unserer Kontrolle ist (lokales dnsmasq oder in einem opnsense router).

### Links

- Anleitung
  - <https://access.redhat.com/documentation/de-de/red_hat_quay/2.9/html/deploy_red_hat_quay_-_basic/installing_red_hat_quay_basic>

## troubleshooting

```bash
 ⠋ quay-db Pulling                                                                                                    2.1s
   ⠇ 5181eccc7c90 Downloading [>                                                  ]       ...                         0.8s
Error response from daemon: failed to resolve reference "quay.io/project-quay/quay:3.8.5": pulling from host quay.io failed with status code [manifests 3.8.5]: 401 UNAUTHORIZED
```

- login to quay with
  - `docker login -u="redhat+quay" -p="O81WSHRSJR14UAZBK54GQHJS0P1V4CLWAJV1X2C4SD7KO59CQ9N3RE12612XU1HR" quay.io`

- see  <https://access.redhat.com/solutions/3533201>

### Beim multi-arc build schlägt push auf die Registry fehl mit 127.0.0.1:443: connect: connection refused

Das kann nur passieren, wenn man die Ziel-Registry selber aufgesetzt hat und lokal unter der localhost Adresse betreibt.

Problem ist hier, dass Docker im Docker-Container baut und dabei auf die localhost Adresse zugreift. Das funktioniert nicht, weil der Docker-Container nicht auf die localhost Adresse zugreift, sondern auf die IP-Adresse des Docker-Hosts.

```bash
ERROR: failed to solve: failed to push quay.localhost.direct/admin/template-ausbildung:1.0: failed to do request: Head "https://quay.localhost.direct/v2/admin/template-ausbildung/blobs/sha256:73a0f73e05f3bccd63841c25447c01519d92c0221ed1858392943842ca196ea5": dial tcp 127.0.0.1:443: connect: connection refused
```

### multi architecture build bricht beim push ab

- im quay log findet sich `jsonschema.exceptions.ValidationError: 'application/vnd.in-toto+json' is not one of [...]`
- vollständiger log Auszug:

    ```bash
    quay        | gunicorn-registry stdout | 2023-04-24 11:12:37,996 [235] [ERROR] [endpoints.v2.manifest] failed to parse manifest when writing by tagname
    quay        | gunicorn-registry stdout | Traceback (most recent call last):
    quay        | gunicorn-registry stdout |   File "/quay-registry/image/oci/manifest.py", line 155, in __init__
    quay        | gunicorn-registry stdout |     validate_schema(self._parsed, self.get_meta_schema(ignore_unknown_mediatypes))
    quay        | gunicorn-registry stdout |   File "/app/lib/python3.9/site-packages/jsonschema/validators.py", line 934, in validate
    quay        | gunicorn-registry stdout |     raise error
    quay        | gunicorn-registry stdout | jsonschema.exceptions.ValidationError: 'application/vnd.in-toto+json' is not one of ['application/vnd.oci.image.layer.v1.tar', 'application/vnd.oci.image.layer.v1.tar+gzip', 'application/vnd.oci.image.layer.v1.tar+zstd', 'application/vnd.oci.image.layer.nondistributable.v1.tar', 'application/vnd.oci.image.layer.nondistributable.v1.tar+gzip', 'application/vnd.oci.image.layer.v1.tar+zstd', 'application/vnd.sylabs.sif.layer.v1+tar', 'application/tar+gzip', 'application/vnd.cncf.helm.chart.content.v1.tar+gzip']
    quay        | gunicorn-registry stdout | Failed validating 'enum' in schema['properties']['layers']['items']['properties']['mediaType']:
    quay        | gunicorn-registry stdout |     {'description': 'The MIME type of the referenced manifest',
    quay        | gunicorn-registry stdout |      'enum': ['application/vnd.oci.image.layer.v1.tar',
    quay        | gunicorn-registry stdout |               'application/vnd.oci.image.layer.v1.tar+gzip',
    quay        | gunicorn-registry stdout |               'application/vnd.oci.image.layer.v1.tar+zstd',
    quay        | gunicorn-registry stdout |               'application/vnd.oci.image.layer.nondistributable.v1.tar',
    quay        | gunicorn-registry stdout |               'application/vnd.oci.image.layer.nondistributable.v1.tar+gzip',
    quay        | gunicorn-registry stdout |               'application/vnd.oci.image.layer.v1.tar+zstd',
    quay        | gunicorn-registry stdout |               'application/vnd.sylabs.sif.layer.v1+tar',
    quay        | gunicorn-registry stdout |               'application/tar+gzip',
    quay        | gunicorn-registry stdout |               'application/vnd.cncf.helm.chart.content.v1.tar+gzip'],
    quay        | gunicorn-registry stdout |      'pattern': '\\w+/[-.\\w]+(?:\\+[-.\\w]+)?',
    quay        | gunicorn-registry stdout |      'type': 'string'}
    quay        | gunicorn-registry stdout | On instance['layers'][0]['mediaType']:
    quay        | gunicorn-registry stdout |     'application/vnd.in-toto+json'
    ```

- Das Problem ist bekannt und kann leicht behoben werden (ist auch schon behoben in diesem repository in der [services/config.yaml](./services/config.yaml) Datei))

  - dieser Abschnitt:

      ```yaml
      ALLOWED_OCI_ARTIFACT_TYPES:
          application/vnd.oci.image.config.v1+json:
              - application/vnd.oci.image.layer.v1.tar+zstd
          application/vnd.sylabs.sif.config.v1+json:
              - application/vnd.sylabs.sif.layer.v1+tar
      ```

  - wird ergänzt um `- application/vnd.in-toto+json`

      ```yaml
      ALLOWED_OCI_ARTIFACT_TYPES:
          application/vnd.oci.image.config.v1+json:
              - application/vnd.oci.image.layer.v1.tar+zstd
              - application/vnd.in-toto+json
          application/vnd.sylabs.sif.config.v1+json:
              - application/vnd.sylabs.sif.layer.v1+tar
      ```

- see <https://access.redhat.com/solutions/6999695>
