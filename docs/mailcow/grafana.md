Mit der Grafana Cloud bietet Grafana Labs eine moderne und umfangreiche Lösung zum Monitoring.\
Durch die einzelnen Komponentenen die bereits in der Free Edition verfügbar sind, steht so eine umfangreicher Monitoring und Alerting Stack zur Verfügung:
- Grafana
- Prometheus
- Loki
- Grafit
- Alerts
- Tempo

Dieser kann als nützliche Ergänzung für Mailcow Installationen verwendet werden.

Im folgenden werden einige mögliche Szenarien zum Einsatz der Grafana Cloud beschrieben.\
Alle Konfigurationen sind aber natürlich auch mit einem lokalen Grafana Stack möglich.

# Grafana Cloud
TBD
# Logfile Monitoring mit Loki
[Grafana Loki](https://grafana.com/oss/loki/) ist die Log File Analyse Lösung von Grafana.\
Um diese mit Mailcow einsetzen zu können, empfiehlt es sich, vom Loki Plugin für Docker Abstand zu nehmen (bricht Mailcow Funktionen, die auf dem JSON Log Driver von Docker basieren) und stattdessen, die Docker JSON Logs über den Standard JSON Log Driver zu verwenden.\
Als Log Shipping Tool bietet sich hier [Promtail](https://grafana.com/docs/loki/latest/clients/promtail/) als Docker Container an, der als Ergänzung im `docker-compose.override.yml` aufgenommen und damit automatisch vom Mailcow Update Script gepflegt wird.

Um die Logs in Grafan sauber verarbeiten zu können, sollte (optional) der JSON Log Driver um einige Tags erweitert werden. Hier geht es primär um Lesbarkeit (z.B. Container Namen vs. Kryptische Pfade).

## Docker Daemon JSON Log Driver Anpassung
>Alle hier aufgeführten Anpassungen stammen aus diesem [Beitrag](https://gist.github.com/ruanbekker/c6fa9bc6882e6f324b4319c5e3622460?permalink_comment_id=3621878#gistcomment-3621878) von [tuxuser](https://gist.github.com/tuxuser). Herzlichen Dank an Tuxuser!

Um den Docker Daemon anzuweisen die Logs um zusätzliche Tags zu erweitern, muss die Datei `/etc/docker/daemon.json` wie folgt gepatched werden.
```
--- /etc/docker/daemon.json     2022-02-20 15:57:17.156432214 +0100
+++ daemon.json.org     2022-02-21 09:16:23.161162897 +0100
@@ -1 +1 @@
-{"ipv6":true,"fixed-cidr-v6":"fd00:dead:beef:c0::/80","experimental":true,"ip6tables":true,"log-driver": "json-file","log-opts": {"tag": "{{.ImageName}}|{{.Name}}|{{.ImageFullID}}|{{.FullID}}"}}
+{"ipv6":true,"fixed-cidr-v6":"fd00:dead:beef:c0::/80","experimental":true,"ip6tables":true}
```

**Zur Erklärung:**

| Eintrag  | Auswirkung |
| ------------- | ------------- |
| "log-driver": "json-file"  | Referenziert auf den JSON File Log Driver (default)  |
| "log-opts": {"tag": "{{.ImageName}}\|{{.Name}}\|{{.ImageFullID}}\|{{.FullID}}"  | Ergänzt den Eintrag um die Tags `Image Name` = Name des Docker Images, `Name`= Container Name, etc.   |

## Loki Beispiele

### RSPAMD
`{container_name="mailcowdockerized_rspamd-mailcow_1"} |~  "rspamd_task_write_log" | pattern "<_> <_> <_> <_> <_> <_> <_> <_> <_> <_> <_> <IP>,<_> from: <from>, <_> F ( <action> )"`

# Performance Monitoring mit Prometheus
