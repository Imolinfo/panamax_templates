          Memo configurazione progetto Tubo

Breve descrizione dei passi da seguire per la messa in opera dell'applicativo Tubo. 
Si presuppone una conoscenza basilare di Docker.

Requisiti:
- file tubo.war deployato con ambiente di sviluppo Eclipse
- Docker Engine (boot2docker su Windows, Panamax su Linux)

Quattro container:
- Sender, applicativo per l'invio di file
- Receiver, applicativo per la ricezione
- Data sender, repository configurazione per Sender
- Data receiver, repository configurazione per Receiver

Per creare Sender comando Docker:
docker run --name tomcat_2 -p 8081:8080 -e "CATALINA_OPTS=-DTUBO_CONFIGURATION_PATH=/sender/sender.yml" -v /home/core/logs/sender:/usr/local/tomcat/logs --volumes-from "data_sender_2223" tomcat

Per creare Receiver comando Docker:
docker run --name tomcat_1 -p 8083:8080 -e "CATALINA_OPTS=-DTUBO_CONFIGURATION_PATH=/receiver/receiver.yml" -v /home/core/logs/receiver:/usr/local/tomcat/logs --volumes-from "data_receiver_2224" tomcat

Per Data sender:
docker run --name data_sender_2223 -p 2223:22 -v /sender -v /usr/local/tomcat/webapps rastasheep/ubuntu-sshd

Per Data receiver:
docker run --name data_receiver_2224 -p 2224:22 -v /receiver -v /usr/local/tomcat/webapps rastasheep/ubuntu-sshd

Una volta che i container sono in piedi, è necessario caricare i file tubo.war e le relative risorse  di configurazione. Allo scopo scaricare l'archivio panamax_templates dal repository Git e scompattarlo in locale.

Configurazione Data sender
Consiste nell'effettuare l'upload entro il container Data sender delle risorse necessarie. Entrare in config/sender.
Primo passo creare le directory mancanti, il nome è reperibile dal file di configurazione sender.yml:
- outbox
- archive
- outbox.receipts
- errors
- logs
Effettuare upload tramite scp entro container Data sender col comando (si suppone essere entro sender):
scp -r -P 2223 ./* root@10.0.0.200:/sender

Ora caricare il war con  comando:
scp -P 2223 tubo-1.0.2.war root@10.0.0.200:/usr/local/tomcat/webapps

Configurazione Data receiver
Duale a quella di Data sender.  Entarre in config/receiver.
Creare le cartelle mancanti visionando il file receiver.yml:
- inbox
- inbox-receipts
- logs
Da receiver effettuaare upload con:
scp -r -P 2224 ./* root@10.0.0.200:/receiver

Successivo caricamento del war con:
scp -P 2224 tubo-1.0.2.war root@10.0.0.200:/usr/local/tomcat/webapps

Note:
- Credenziali accesso scp root root, già pre-configurate nell'iimagine
- Nel documento usato l'indirizzo 10.0.0.200 valido in ambiente Panamax, nel caso si usasse un ambiente diverso sostituire con l'opportuno ip
- E' possibile accedere ai container tramite ssh, la sintassi del comando: 
  ssh root@10.0.0.200 -p 2224 (o 2223 a seconda si tratti di Receiver o Sender)
