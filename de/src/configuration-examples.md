# Anwendungs- und Konfigurationsbeispiele

Hier werden verschiedene Anwendungsbeispiele von
*raspiBackup* sowie ihrer Konfiguration vorgestellt und erklärt. Sie sollen
helfen, aus der Vielzahl der Anwendungsmöglichkeiten die Richtige zu finden oder
das Beispiel dann nach den eigenen Ansprüchen entsprechend anzupassen. Eine
Übersicht aller Optionen findet sich in [Aufruf und Optionen](usage-and-options.md).
Verschiedene Methoden, ein Backup zu restoren, sind im Kapitel [Wiederherstellen/Restore](restore.md) beschrieben.

Alle Konfigurationen, die kein dd Backup benutzen, sichern auch ein externes
Rootfilesystem zusammen mit einer SD Bootpartition. Wenn der USB Bootmode benutzt
wird und keine SD Karte mehr benutzt wird, wird auch die ganze Rootpartition
gesichert.


Folgende Anwendungsbeispiele werden beschrieben:

<!-- toc -->


## Ein Windowsbenutzer möchte seine Raspberry sichern und auf Windows restoren können.

Um ein Image unter Windows restoren zu können muss ein dd
Image von *raspiBackup* erstellt werden. Folgende Konfigurationsoptionen sind
dazu wenigstens notwendig:

```
DEFAULT_BACKUPTYPE=dd
DEFAULT_KEEPBACKUPS=n
```

## ... Er hat eine 32GB SD Karte und benutzt nur 12GB davon die er aber auch nur sichern möchte

Zusätzlich zu den genannten Optionen ist die folgende Option notwendig:

```
DEFAULT_DD_BACKUP_SAVE_USED_PARTITIONS_ONLY=1
```

Allerdings ist dazu auch notwendig die Rootpartition der Raspberry zu
verkleinern, da standardmäßig der gesamte freien Platz der SD Karte gesichert 
wird. Dieses geht aber nicht unter Windows sondern es muss ein Linux benutzt
werden und mit den Tools gparted oder resize2fs dann die Rootpartition
verkleinert werden.

## Ein Windowsbenutzers möchte mit pishrink ein absolut minimales Image erstellen

Um ein minimalstes Backup zu erzeugen kann man das Tool pishrink benutzen. Dazu
gibt es das Script raspiBackupWrapper.sh mit welchem man am Ende des Backups
das dd Image per pishrink verkleinern kann. Die Option

```
DEFAULT_ZIP_BACKUP=1
```

verkleinert zwar auch noch mal das Image aber das kann nicht direkt 
iunter Windows restored werden. Es muss uvorher unzipped werden.

## Eine Raspberry soll möglichst schnell gesichert werden. Die Backuppartition ist ein per nfs gemountetes EXT4 Dateisystem welches von einer NAS zur Verfügung gestellt wird.

Zuerst muss die Backuppartition der NAS gemounted werden. Dazu sollte in
/etc/fstab die nfs Partition definiert und automatisch unter /backup
gemounted sein.

```
DEFAULT_BACKUPTYPE=rsync
DEFAULT_KEEPBACKUPS=n
```

Da das Backupfilesystem mit EXT4 formatiert ist kann *raspiBackup* Hardlinks
benutzen und das beschleunigt den Backupprozess sehr da nur die geänderten
Dateien gesichert werden.

Ein Beispieleintrag in der /etc/fstab könnte wie folgt aussehen:

```
asterix:/backup    /backup    nfs    users,rw,sync,hard,intr,noauto,user    0    0
```

Dabei ist asterix der hostname der NAS und /backup der exportierte nfs
Mount. Weitere Hinweise zu Synology spezifischen Einstellungen und
Problemlösungen finden sich [hier](nfs-as-backupspace.md)


## Eine Raspberry soll auf ein per SMB gemountetes Dateisystem gesichert werden, welches von einem Windowssystem zur Verfügung gestellt wird.

```
DEFAULT_BACKUPTYPE=tar
DEFAULT_KEEPBACKUPS=n
```

Das remote Windows Backupfilesystem sollte in der /etc/fstab definiert sein und
automatisch gemounted werden. Es wird jedes Mal das gesamte iSystem gesichert.
Dabei ist darauf zu achten dass das Filesystem auf dem SMB Laufwerk Dateien >
4GB unterstützen muss, denn die tar Dateien sind i.d.R. > 4GB. FAT32 reicht
dazu nicht. Siehe dazu auch [Welches Filesystem kann man auf der Backuppartition nutzen](which-filesystem-can-be-used-on-the-backup-partition.md)

Ein Beispieleintrag in der /etc/fstab könnte wie folgt aussehen:

```
//asterix/backup/ /backup    cifs noauto,noatime,user,utf8,umask=000,uid=1000,gid=1000,credentials=/etc/samba/auth.asterix.cifsuser 0 0
```

## Es ist eine größere Änderung an der Raspberry beabsichtigt und verschiedene Zwischenstände sollen sicherheitshalber gesichert werden.

Hierzu ist eine fertige Konfiguration von *raspiBackup* erforderlich (siehe
vorhergehende Beispiele). Dann muss *raspiBackup* nur noch mit der Aufrufoption

```
-m "Das ist ein sprechender Name des Backups"
```

aufgerufen werden und es wird ein Backup mit genau diesem sprechenden Namen im Backupverzeichnis /backup erstellt.

**Hinweis**: Dieses Backup wird nicht beim Backuprecycle berücksichtigt und muss manuell gelöscht werden.

## Ein USB Boot System soll mit weiteren Partitionen gesichert werden.

In diesem Falle muss der partitionsorientierte Backup gewählt werden sowie die
zu sichernden Partitionen konfiguriert werden. Im Beispiel soll die Partition 5
noch mitgesichert werden.

```
DEFAULT_PARTITIONBASED_BACKUP=1
DEFAULT_BACKUPTYPE=rsync
DEFAULT_KEEPBACKUPS=n
DEFAULT_PARTITIONS_TO_BACKUP="1 2 5"
```


## Eine Raspberry soll auf einen lokal angeschlossen USB Stick oder eine lokal angeschlossene USB Platte gesichert werden.

```
DEFAULT_BACKUPTYPE=rsync
DEFAULT_KEEPBACKUPS=n
DEFAULT_BACKUPPATH="/USBStick"
```

Damit rsync Hardlinks benutzt und der Backup schnell ist muss die
Backuppartition mit ext3/4 formatiert sein. Will man Daten mit Windows
austauschen und die Partition wurde mit Windows formatiert ist tar als
Backuptype zu benutzen. Dann dauert aber das Backup länger und das Backup
benötigt viel mehr Platz.

**Hinweis**: Falls die USB Partition von Windows aus zugreifbar sein soll muss sie mit NTFS formatiert sein.
Dann ist aber kein Backuptyp rsync möglich. NTFS kann nur mit dem Backuptype `dd` und `tar` genutzt werden und
der DEFAULT_BACKUPTYPE muss dann entsprechend gesetzt werden.

Ein Beispieleintrag in der /etc/fstab könnte wie folgt aussehen:

```
LABEL=usb    /USBStick    ext4     defaults,noatime,nofail        0    2
```

[.status]: rft
[.source]: https://linux-tips-and-tricks.de/de/konfigurationsbeispiele
[.source]: https://www.linux-tips-and-tricks.de/en/configuration-samples

