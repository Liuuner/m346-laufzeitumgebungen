# M346 - Laufzeitumgebungen

---

## Laufendes Projekt

Das Projekt läuft auf
[m347.liuuner.ch](https://m347.liuuner.ch)

## Planung
Wir haben uns für Linode entschieden, da ich darauf schon einen Kubernetes Cluster aufgesetzt hatte und es nicht so teuer ist wie Digital Ocean.

## Umsetzung
### Dockerfiles
Ich musste die Dockerfiles noch anpassen da sie auf meiner Maschine nicht funktionierten. Warscheinlich weil die vorherigen Images auf ARM basierten.

**Backend**
Das Backend wird mit diesem [Dockerfile](backend/Dockerfile) gebaut. 
Ich änderte die Applikation noch ab damit ich die Configs über Environment Variables einspeisen kann.

**Frontend**
Ich änderte das [Dockerfile](frontend/Dockerfile) ab damit ich einen Multistage Build habe um die Image Size zu minimieren.

### Lokal
Lokal konnte ich die Applikation mit einem simplen [docker-compose](docker-compose.yml) File starten und es funktionierte einwandfrei.

### Cloud
Wie schon erwähnt machten wir es mit Kubernetes.
Das ermöglichte es uns, schnell neue Versionen der Images auszuprobieren ohne dass wir viele Schritte machen mussten.
Für das Frontend sowie Backend erstellten wir ein Deployment und einen Service.
Das Backend brauchte zusätzlich noch eine ConfigMap für die Datenbank Connection Infos und ein Secret um die Datenbank Credentials sicher aufzubewahren.

**Frontend:**

[deployment](k8s/frontend/deployment.yaml)
[service](k8s/frontend/service.yaml)

**Backend:**

[deployment](k8s/backend/deployment.yaml)
[service](k8s/backend/service.yaml)
[configmap](k8s/configmap.yaml)
[credentials](k8s/credentials.yaml)

**Datenbank:**

Die Datenbank erstellte ich über eine OneClick Installation in Linode.
Für unsere Applikation verwendeten wir MySQL.
Es funktionierte nicht bis ich herausgefunden habe, dass man den externen zugriff zuerst noch aktivieren muss.
Dieses Problem konnte ich mit dem [Tutorial von DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-allow-remote-access-to-mysql)bewältigen.

### CI/CD
#### Github-Actions
Ich wollte dass nur dann ein Build und Push gemacht wird wenn ich einen Release tätige. Damit stimmen die Versionsnummern immer klar mit dem Codestand der Version überein.
Dafür schrieb ich einen [Release Workflow](.github/workflows/release.yaml) welcher:
- Die nächste Versionsnummer berechnet auf basis vom input { Major, Minor, Patch }
- Ein bestimmter Commit Hash mit der Versionsnummer taggt
- Eine Release Page mit dem Changelog generiert
Die nächste Versionsnummer wird mit der [SemVers Action](.github/actions/semVers/action.yml) berechnet welche eigentlich nur ein Bash Script ist.
Um die Release Page zu generieren musste ich einen PersonalAccessToken erstellen. Der Grund wird später im Build erklärt.

Als nächstes musste ich Build und Push ermöglichen.
Es gibt schon viele Wiederverwendbare Actions von Docker selbst welche einzelne Teilschritte wie Login und den Build übernehmen. Für das Login musste ich meine Login Credentials angeben welche in als Secret sicher abgespeichert hatte.
Ich machte es so dass der Workflow durch einen Release getriggert wird.
Und hier ist auch der Grund warum ich beim Release erstellen einen PAT brauchte.
Im Workflow nutzte ich den folgenden Trigger.
```yaml
on:
  release:
    types:
      - published
```
Dieser wird aber nur getriggert wenn der Release nicht automatisch erstellt wird.
Das habe ich nach langem Kopfzerbrechen durch die [Diskussion](https://github.com/orgs/community/discussions/25281) auf Github herausgefunden.
Umgehen konnte ich es mit einem PAT welcher imitiert dass ich den Release getätigt hätte.
Als ich das herausgefunden hatte war der [Build Workflow](.github/workflows/build.yaml) fertig.

#### ArgoCD
Deployed wird es auf den Server mithilfe von ArgoCD. 
Wenn man in den [Resource-Definitions](k8s) die Version des Docker-Images ändert, bekommt das ArgoCD mit und deployed die neue Version des Containers automatisch.
Das ist alles möglich dank Kubernetes.


## Arbeitsaufteilung
Es gab keine wirkliche Arbeitsaufteilung da wir zuerst alle miteinander versuchten, die Builds zum laufen zu bringen was lange dauerte.
Und danach musste man auch nicht mehr so viel machen.

## Reflexion
Die Umsetzung des Projekts _M346 - Laufzeitumgebungen_ verlief erfolgreich und lehrreich. Durch die Wahl von Linode und den bereits bestehenden Kubernetes Cluster konnten wir kostengünstig und effizient arbeiten. Die Anpassungen an den Dockerfiles und die Konfiguration der Deployments waren sehr aufwändig um das Problem zu ermitteln doch wir haben es schlussendlich geschafft. Die Integration von CI/CD-Pipelines mit GitHub Actions und ArgoCD ermöglichte eine nahtlose und automatisierte Bereitstellung neuer Container-Versionen.