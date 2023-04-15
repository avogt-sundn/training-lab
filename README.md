# Wir bauen uns eine Labor-Umgebung

## Quay - eine eigene Docker registry

Ziel:
- installieren von quay mittels docker compose


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