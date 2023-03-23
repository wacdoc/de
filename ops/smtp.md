Geben Sie das Konfigurations-Warehouse ops.soft ein, führen Sie `./ssl.sh` aus, und ein `conf` Ordner wird **im oberen Verzeichnis** erstellt.

## Präambel

SMTP kann Dienste direkt von Cloud-Anbietern erwerben, wie z. B.:

* [Amazon SES-SMTP](https://docs.aws.amazon.com/ses/latest/dg/send-email-smtp.html)
* [Ali Cloud-E-Mail-Push](https://www.alibabacloud.com/help/directmail/latest/three-mail-sending-methods)

Sie können auch Ihren eigenen Mailserver aufbauen - unbegrenztes Senden, geringe Gesamtkosten.

Im Folgenden zeigen wir Schritt für Schritt, wie Sie Ihren eigenen Mailserver erstellen.

## Serverauswahl

Der selbst gehostete SMTP-Server erfordert eine öffentliche IP mit offenen Ports 25, 456 und 587.

Häufig verwendete öffentliche Clouds haben diese Ports standardmäßig blockiert, und es ist möglicherweise möglich, sie durch Erteilen eines Arbeitsauftrags zu öffnen, aber es ist schließlich sehr mühsam.

Ich empfehle den Kauf bei einem Host, der diese Ports offen hat und die Einrichtung von Reverse-Domainnamen unterstützt.

Hier empfehle ich [Contabo](https://contabo.com) .

Contabo ist ein Hosting-Anbieter mit Sitz in München, Deutschland, der 2003 mit sehr wettbewerbsfähigen Preisen gegründet wurde.

Wenn Sie Euro als Kaufwährung wählen, ist der Preis günstiger (ein Server mit 8 GB Speicher und 4 CPUs kostet etwa 529 Yuan pro Jahr, und die anfängliche Installationsgebühr ist für ein Jahr kostenlos).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/UoAQkwY.webp)

Wenn Sie eine Bestellung aufgeben, erwähnen Sie `prefer AMD` , und der Server mit AMD-CPU wird eine bessere Leistung haben.

Im Folgenden nehme ich den VPS von Contabo, um zu demonstrieren, wie man einen eigenen Mailserver aufbaut.

## Ubuntu-Systemkonfiguration

Das Betriebssystem ist hier Ubuntu 22.04

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/smpIu1F.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/m7Mwjwr.webp)

Wenn der Server auf ssh `Welcome to TinyCore 13!` (wie in der Abbildung unten gezeigt), bedeutet dies, dass das System noch nicht installiert wurde. Bitte trennen Sie ssh und warten Sie einige Minuten, um sich erneut anzumelden.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/-qKACz9.webp)

Wenn `Welcome to Ubuntu 22.04.1 LTS` angezeigt wird, ist die Initialisierung abgeschlossen und Sie können mit den folgenden Schritten fortfahren.

### [Optional] Initialisieren Sie die Entwicklungsumgebung

Dieser Schritt ist optional.

Der Einfachheit halber habe ich die Installation und Systemkonfiguration der Ubuntu-Software in [github.com/wactax/ops.os/tree/main/ubuntu](https://github.com/wactax/ops.os/tree/main/ubuntu) abgelegt.

Führen Sie den folgenden Befehl aus, um mit einem Klick zu installieren.

```
bash <(curl -s https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

Chinesische Benutzer verwenden bitte stattdessen den folgenden Befehl, und die Sprache, Zeitzone usw. wird automatisch eingestellt.

```
CN=1 bash <(curl -s https://ghproxy.com/https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

### Contabo ermöglicht IPV6

Aktivieren Sie IPV6, damit SMTP auch E-Mails mit IPV6-Adressen versenden kann.

Bearbeiten Sie `/etc/sysctl.conf`

Ändern oder fügen Sie die folgenden Zeilen hinzu

```
net.ipv6.conf.all.disable_ipv6 = 0
net.ipv6.conf.default.disable_ipv6 = 0
net.ipv6.conf.lo.disable_ipv6 = 0
```

Folgen Sie [dem Contabo-Tutorial: IPv6-Konnektivität zu Ihrem Server hinzufügen](https://contabo.com/blog/adding-ipv6-connectivity-to-your-server/)

Bearbeiten Sie `/etc/netplan/01-netcfg.yaml` , fügen Sie ein paar Zeilen hinzu, wie in der Abbildung unten gezeigt (Contabo VPS-Standardkonfigurationsdatei enthält diese Zeilen bereits, kommentieren Sie sie einfach aus).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/5MEi41I.webp)

Dann `netplan apply` , damit die geänderte Konfiguration wirksam wird.

Nach erfolgreicher Konfiguration können Sie mit `curl 6.ipw.cn` die IPv6-Adresse Ihres externen Netzwerks anzeigen.

## Klonen Sie die Operationen des Konfigurations-Repositorys

```
git clone https://github.com/wactax/ops.soft.git
```

## Generieren Sie ein kostenloses SSL-Zertifikat für Ihren Domainnamen

Das Versenden von E-Mails erfordert ein SSL-Zertifikat zum Verschlüsseln und Signieren.

Wir verwenden [acme.sh](https://github.com/acmesh-official/acme.sh) , um Zertifikate zu generieren.

acme.sh ist ein Open-Source-Tool zum automatischen Signieren von Zertifikaten.

Geben Sie das Konfigurations-Warehouse ops.soft ein, führen Sie `./ssl.sh` aus, und ein `conf` Ordner wird **im oberen Verzeichnis** erstellt.

Suchen Sie Ihren DNS-Anbieter in [acme.sh dnsapi](https://github.com/acmesh-official/acme.sh/wiki/dnsapi) , bearbeiten Sie `conf/conf.sh` .

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Qjq1C1i.webp)

Führen Sie dann `./ssl.sh 123.com` aus, um `123.com` und `*.123.com` Zertifikate für Ihren Domänennamen zu generieren.

Bei der ersten Ausführung wird automatisch [acme.sh](https://github.com/acmesh-official/acme.sh) installiert und eine geplante Aufgabe für die automatische Erneuerung hinzugefügt. Sie können `crontab -l` sehen, es gibt eine solche Zeile wie folgt.

```
52 0 * * * "/mnt/www/.acme.sh"/acme.sh --cron --home "/mnt/www/.acme.sh" > /dev/null
```

Der Pfad für das generierte Zertifikat ist etwa `/mnt/www/.acme.sh/123.com_ecc。`

Die Zertifikatserneuerung ruft das Skript `conf/reload/123.com.sh` auf, bearbeiten Sie dieses Skript, Sie können Befehle wie `nginx -s reload` hinzufügen, um den Zertifikatcache verwandter Anwendungen zu aktualisieren.

## Erstellen Sie einen SMTP-Server mit chasquid

[chasquid](https://github.com/albertito/chasquid) ist ein Open-Source-SMTP-Server, der in der Go-Sprache geschrieben ist.

Als Ersatz für die alten Mailserver-Programme wie Postfix und Sendmail ist chasquid einfacher und benutzerfreundlicher und auch für die sekundäre Entwicklung einfacher.

Führen Sie `./chasquid/init.sh 123.com` wird automatisch mit einem Klick installiert (ersetzen Sie 123.com durch Ihren sendenden Domainnamen).

## E-Mail-Signatur DKIM konfigurieren

DKIM wird zum Senden von E-Mail-Signaturen verwendet, um zu verhindern, dass Briefe als Spam behandelt werden.

Nachdem der Befehl erfolgreich ausgeführt wurde, werden Sie aufgefordert, den DKIM-Eintrag festzulegen (wie unten gezeigt).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/LJWGsmI.webp)

Fügen Sie Ihrem DNS einfach einen TXT-Eintrag hinzu (wie unten gezeigt).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/0szKWqV.webp)

## Dienststatus und Protokolle anzeigen

 `systemctl status chasquid` Dienststatus anzeigen.

Der normale Betriebszustand ist in der folgenden Abbildung dargestellt

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/CAwyY4E.webp)

 `grep chasquid /var/log/syslog` oder `journalctl -xeu chasquid` können das Fehlerprotokoll anzeigen.

## Umgekehrte Domänennamenkonfiguration

Der umgekehrte Domainname soll die Auflösung der IP-Adresse in den entsprechenden Domainnamen ermöglichen.

Das Festlegen eines Reverse-Domainnamens kann verhindern, dass E-Mails als Spam identifiziert werden.

Wenn die E-Mail empfangen wird, führt der empfangende Server eine Reverse-Domain-Namensanalyse für die IP-Adresse des sendenden Servers durch, um zu bestätigen, ob der sendende Server über einen gültigen Reverse-Domain-Namen verfügt.

Wenn der sendende Server keinen Reverse-Domain-Namen hat oder der Reverse-Domain-Name nicht mit der IP-Adresse des sendenden Servers übereinstimmt, kann der empfangende Server die E-Mail als Spam erkennen oder ablehnen.

Besuchen Sie [https://my.contabo.com/rdns](https://my.contabo.com/rdns) und konfigurieren Sie wie unten gezeigt

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/IIPdBk_.webp)

Denken Sie nach dem Festlegen des umgekehrten Domänennamens daran, die Vorwärtsauflösung des Domänennamens ipv4 und ipv6 zum Server zu konfigurieren.

## Bearbeiten Sie den Hostnamen von chasquid.conf

Ändern Sie `conf/chasquid/chasquid.conf` auf den Wert des umgekehrten Domänennamens.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/6Fw4SQi.webp)

Führen Sie dann `systemctl restart chasquid` aus, um den Dienst neu zu starten.

## Sichern Sie die conf im Git-Repository

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Fier9uv.webp)

Zum Beispiel sichere ich den conf-Ordner wie folgt in meinem eigenen Github-Prozess

Erstellen Sie zuerst ein privates Lager

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ZSCT1t1.webp)

Geben Sie das conf-Verzeichnis ein und senden Sie es an das Warehouse

```
git init
git add .
git commit -m "init"
git branch -M main
git remote add origin git@github.com:wac-tax-key/conf.git
git push -u origin main
```

## Absender hinzufügen

laufen

```
chasquid-util user-add i@wac.tax
```

Kann Absender hinzufügen

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/khHjLof.webp)

### Stellen Sie sicher, dass das Passwort richtig eingestellt ist

```
chasquid-util authenticate i@wac.tax --password=xxxxxxx
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/e92JHXq.webp)

Nach dem Hinzufügen des Benutzers wird `chasquid/domains/wac.tax/users` aktualisiert, denken Sie daran, es an das Warehouse zu senden.

## DNS SPF-Eintrag hinzufügen

SPF ( Sender Policy Framework ) ist eine E-Mail-Verifizierungstechnologie zur Verhinderung von E-Mail-Betrug.

Es überprüft die Identität eines E-Mail-Absenders, indem es prüft, ob die IP-Adresse des Absenders mit den DNS-Einträgen des Domainnamens übereinstimmt, der es vorgibt zu sein, und verhindert, dass Betrüger gefälschte E-Mails senden.

Das Hinzufügen von SPF-Einträgen kann so weit wie möglich verhindern, dass E-Mails als Spam identifiziert werden.

Wenn Ihr Domain Name Server den SPF-Typ nicht unterstützt, fügen Sie einfach den TXT-Typ-Eintrag hinzu.

Der SPF von `wac.tax` sieht beispielsweise wie folgt aus

`v=spf1 a mx include:_spf.wac.tax include:_spf.google.com ~all`

SPF für `_spf.wac.tax`

`v=spf1 a:smtp.wac.tax ~all`

Beachten Sie, dass ich hier `include:_spf.google.com` habe, weil ich `i@wac.tax` später als Absenderadresse im Google-Postfach konfigurieren werde.

## DNS-Konfiguration DMARC

DMARC ist die Abkürzung für (Domain-based Message Authentication, Reporting & Conformance).

Es wird verwendet, um SPF-Bounces zu erfassen (möglicherweise verursacht durch Konfigurationsfehler oder jemand anderes gibt vor, Sie zu sein, um Spam zu senden).

TXT-Eintrag hinzufügen `_dmarc` ,

Der Inhalt ist wie folgt

```
v=DMARC1; p=quarantine; fo=1; ruf=mailto:ruf@wac.tax; rua=mailto:rua@wac.tax
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/k44P7O3.webp)

Die Bedeutung der einzelnen Parameter ist wie folgt

### p (Richtlinie)

Gibt an, wie mit E-Mails umgegangen wird, die die SPF- (Sender Policy Framework) oder DKIM- (DomainKeys Identified Mail) Überprüfung nicht bestehen. Der p-Parameter kann auf einen von drei Werten eingestellt werden:

* none: Es wird keine Aktion durchgeführt, nur das Überprüfungsergebnis wird über den E-Mail-Berichtsmechanismus an den Absender zurückgesendet.
* Quarantäne: Legen Sie die E-Mail, die die Überprüfung nicht bestanden hat, in den Spam-Ordner, lehnt die E-Mail jedoch nicht direkt ab.
* Ablehnen: E-Mails, bei denen die Überprüfung fehlschlägt, direkt ablehnen.

### fo (Fehleroptionen)

Gibt die Menge der vom Berichtsmechanismus zurückgegebenen Informationen an. Er kann auf einen der folgenden Werte eingestellt werden:

* 0: Validierungsergebnisse für alle Nachrichten melden
* 1: Nur Nachrichten melden, bei denen die Überprüfung fehlschlägt
* d: Melden Sie nur Fehler bei der Überprüfung des Domänennamens
* s: Nur Fehler bei der SPF-Überprüfung melden
* l: Nur DKIM-Verifizierungsfehler melden

### rua & ruf

* rua (Berichts-URI für aggregierte Berichte): E-Mail-Adresse zum Empfangen aggregierter Berichte
* ruf (Reporting URI for Forensic Reports): E-Mail-Adresse, um detaillierte Berichte zu erhalten

## Fügen Sie MX-Einträge hinzu, um E-Mails an Google Mail weiterzuleiten

Da ich kein kostenloses Unternehmenspostfach finden konnte, das universelle Adressen unterstützt (Catch-All, kann alle E-Mails empfangen, die an diesen Domainnamen gesendet werden, ohne Einschränkungen bei Präfixen), habe ich chasquid verwendet, um alle E-Mails an mein Gmail-Postfach weiterzuleiten.

**Wenn Sie Ihr eigenes kostenpflichtiges Geschäftspostfach haben, ändern Sie bitte nicht den MX und überspringen Sie diesen Schritt.**

Bearbeiten Sie `conf/chasquid/domains/wac.tax/aliases` , legen Sie das Weiterleitungspostfach fest

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/OBDl2gw.webp)

`*` gibt alle E-Mails an, `i` ist das Präfix der E-Mail-Adresse des oben erstellten sendenden Benutzers. Um E-Mails weiterzuleiten, muss jeder Benutzer eine Zeile hinzufügen.

Fügen Sie dann den MX-Eintrag hinzu (ich zeige hier direkt auf die Adresse des Reverse-Domain-Namens, wie in der ersten Zeile in der folgenden Abbildung gezeigt).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/7__KrU8.webp)

Nachdem die Konfiguration abgeschlossen ist, können Sie andere E-Mail-Adressen verwenden, um E-Mails an `i@wac.tax` und `any123@wac.tax` zu senden, um zu sehen, ob Sie E-Mails in Gmail empfangen können.

Wenn nicht, überprüfen Sie das Chasquid-Protokoll ( `grep chasquid /var/log/syslog` ).

## Senden Sie mit Google Mail eine E-Mail an i@wac.tax

Nachdem Google Mail die Mail erhalten hatte, hoffte ich natürlich, mit `i@wac.tax` statt mit i.wac.tax@gmail.com zu antworten.

Besuchen Sie [https://mail.google.com/mail/u/1/#settings/accounts](https://mail.google.com/mail/u/1/#settings/accounts) und klicken Sie auf „Weitere E-Mail-Adresse hinzufügen“.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/PAvyE3C.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/_OgLsPT.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/XIUf6Dc.webp)

Geben Sie dann den Bestätigungscode ein, den Sie von der weitergeleiteten E-Mail erhalten haben.

Schließlich kann es als Standard-Absenderadresse festgelegt werden (zusammen mit der Option, mit derselben Adresse zu antworten).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/a95dO60.webp)

Auf diese Weise haben wir die Einrichtung des SMTP-Mailservers abgeschlossen und verwenden gleichzeitig Google Mail zum Senden und Empfangen von E-Mails.

## Senden Sie eine Test-E-Mail, um zu überprüfen, ob die Konfiguration erfolgreich ist

Geben Sie `ops/chasquid` ein

Führen Sie `direnv allow` aus, um Abhängigkeiten zu installieren (direnv wurde im vorherigen Ein-Tasten-Initialisierungsprozess installiert und der Shell wurde ein Hook hinzugefügt).

dann renne

```
user=i@wac.tax pass=xxxx to=iuser.link@gmail.com ./sendmail.coffee
```

Die Bedeutung der Parameter ist wie folgt

* Benutzer: SMTP-Benutzername
* pass: SMTP-Passwort
* an: Empfänger

Sie können eine Test-E-Mail senden.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ae1iWyM.webp)

Es wird empfohlen, Gmail zu verwenden, um Test-E-Mails zu erhalten, um zu überprüfen, ob die Konfigurationen erfolgreich sind.

### TLS-Standardverschlüsselung

Wie in der Abbildung unten gezeigt, gibt es dieses kleine Schloss, was bedeutet, dass das SSL-Zertifikat erfolgreich aktiviert wurde.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/SrdbAwh.webp)

Klicken Sie dann auf „Original-E-Mail anzeigen“

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/qQQsdxg.webp)

### DKIM

Wie in der Abbildung unten gezeigt, zeigt die ursprüngliche E-Mail-Seite von Gmail DKIM an, was bedeutet, dass die DKIM-Konfiguration erfolgreich war.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/an6aXK6.webp)

Überprüfen Sie das Received im Header der ursprünglichen E-Mail, und Sie können sehen, dass die Absenderadresse IPV6 ist, was bedeutet, dass IPV6 ebenfalls erfolgreich konfiguriert wurde.
