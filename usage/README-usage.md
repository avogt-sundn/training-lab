# Benutzung der labor-Umgebung

## npm registry mirror

Die labor-Umgebung verwendet einen npm registry mirror, der auf dem labor-Server läuft. Dieser ist nur für die labor-Umgebung verfügbar und kann nicht von außen erreicht werden. Um die labor-Umgebung zu nutzen, muss der npm registry mirror konfiguriert werden.

### Konfiguration

Die Konfiguration des npm registry mirrors erfolgt über die Datei `~/.npmrc`. Diese Datei muss folgende Zeilen enthalten:

    registry=http://labor:4873/
    always-auth=true