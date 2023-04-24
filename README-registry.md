# Quay - die Docker registry

Wir lassen die Quay Registry unter https://quay.localhost.direct laufen.

## Einsatz beim docker build - push auf die Quay Registry

```bash
docker buildx build --platform linux/amd64,linux/arm64,linux/arm/v7 -t quay.localhost.direct/admin/template-ausbildung:1.0 --push .

```
docker push

## EInsatz beim build eines VSCode Devcontainers mit dem devcontainer-CLI

-----
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

### docker manifest zeigt nicht die multi-architectures an sondern den Fehler `unsupported manifest media type`

- Fehler bei `docker manifest`
-
    ```bash
    docker manifest inspect quay.localhost.direct/admin/template-ausbildung:1.0
    # unsupported manifest media type and no default available: #   application/vnd.oci.image.manifest.v1+json
    ```
