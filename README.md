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