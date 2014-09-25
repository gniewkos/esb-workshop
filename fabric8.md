Cześć 2 - Fabric8
==========================

## Krok 1 - Uruchomienie Fabric8 ##

Uruchomienie Fabric8:

    cd ~/Programs/fabric8-karaf-1.2.0.Beta4/

Po uruchomieniu, pod poniższym adresem dostępna jest konsola web.

    http://localhost:8181/hawtio

Rozejrzyjmy się po konsoli ssh oraz web. Funkcje dostępne z konsoli web dostępne są także przez ssh. W konsoli ssh dostępne są polecenia znane z ServiceMix oraz dochodzą polecenia z przedrostkiem fabric (fabric:`tab`).

    Fabric8:karaf@root> fabric:container-list

Taką samą listę kontenerów wyświetlamy w konsoli web przez wybranie perspektywy Fabric i zakładki runtime.

## Krok 2 - Uruchomienie quickstart ##

Fabric8 zawiera kilka gotowych do uruchomienia przykładów. Uruchomimy nowy kontener gdzie zainstalujemy przykład camel.cxf.contract.first.

    Fabric8 -> Wiki -> quickstarts -> karaf -> cxf > quickstarts-karaf-cxf -> New -> Container Name: intance01 -> Create and start container

Domyślnie konsola wyświetla widok z perspektywy Fabric8 (czyli cały klaster).
Kilka wskazówek pomagających zrozumieć działanie konsoli:

* Kontener instance01 powinien być widoczny na liście w zakładce Runtime. 
* W zakładce APIs możemy odszukać adres usługi quickstarts-karaf-cxf. 
* Klikając nazwę kontenera przochodzimy do jego szczegółów.
* Klikając strzałkę przy nazwie otwieramy nowe okno z widokiem perspektywy kontenera instance01.
* Perpektywę kontenera możemy zmienić także przełącznikiem po lewej stronie menu głównego.  

Aby sprawdzić działanie usługi możemy wygenerować żądanie w SoapUI i wysłać żądanie.

    SoapUI -> File -> New SOAP Project -> Project Name: Fabric8, Initial WSDL: http://localhost:8183/cxf/order/?wsdl -> OK

W zakładce Camel możemy obejrzeć diagram przepływu na trasach route1 i route2. 
W zakładce Logs możemy obejrzeć logi (niestety przykład nic nie loguje).

Usuwamy kontener z przykładem.

    Fabric8 -> Runtime -> instance01 -> Stop -> Delete

## Krok 3 - Debugowanie ##

Ciekawą funkcjonalnością jest debugowanie trasy integracyjnej. Przetestujemy to na ciekawszym przykładzie angażującym brokera ActiveMQ i wzorzec Content Base Router (CBR).

Uruchamiamy nową instancję brokera ActiveMQ.

    Fabric8 -> Runtime -> MQ -> Create broker configuration -> Group: mygroup, Broker name:mybroker, Kind: StandAlone - > Create

Należy chwilę poczekać na odświeżenie strony z utworzoną grupa i profilem mybroker. Klikamy w ikonę wykrzyknika i dodajemy nowy kontener z tym profilem.
Utworzona konfiguracja to dwa profile:

* Mq/Broker/mygroup.mybroker - służy do uruchomienia Brokera
* Mq/Client/mygroup

*Profil opisuje co zainstalować, jak to skonfigurować, jakie zależności muszą być spełnione a nawet jakie są wymagania dla dostępności usługi.*

    Container Name: instancemq -> Create and start container

Zadanie do wykonania to uruchomienie przykładu **quickstarts-karaf-camel.amq** na nowym kontenerze **instance01**.

Możemy wspomóc się opisem Readme przykładu lub spróbować samodzielnie wykorzystując podpowiedź:
Na kontenerze instance01 powinny zostać uruchomione dwa profile: camel.amq oraz mq/client/mygroup 

Po uruchomieniu kontenera przełączamy perspektywę i przechodzimy do debugowania:

    instance01 -> Camel -> Debug - Enable

Wybieramy trasę jms-cbr-route i dodajemy breakpoint w węzłach `Choice` i `Log` (zaznaczamy węzeł i klikamy ikonę +).

Z listy Endpoints wybieramy `file://work/jms/input`, przechodzimy na zakładkę Send i zakładkę Choose.
Zaznaczamy plik order1.xml i klikamy `Send the file`.

Możemy wrócić do trasy jms-cbr-route gdzie przepływ powinien zatrzymać się na pierwszym breakpoint.
Możemy dokończyć debugowanie.

Po zakończeniu testów usuwamy kontener.

    Fabric8 -> Runtime -> instance01 -> Stop -> Delete

## Krok 4 - instalacja własnej usługi ##

Zainstalujemy naszą usługę w wersji prostego proxy (wracając się do Kroku 8).

W konsoli ssh nadal działa polecenie `install -s`. Jednak w Fabric8 instalujemy paczki za pomocą mechanizmu profili. 
Profil opisuje co zainstalować, jak to skonfigurować, jakie zależności muszą być spełnione a nawet jakie są nasze wymagania dla dostępności usługi.

Profile można tworzyć ręcznie ale możemy także wygenerować je automatycznie podczas budowania projektu. Zróbmy to:

    $ cd ~/Workspace/esb-workshop/report-service
    $ mvn io.fabric8:fabric8-maven-plugin:1.2.0.Beta4:deploy

Powyższe polecenie wygeneruje profil uruchomieniowy naszej usługi i zainstaluje go w Fabric8.

Znajdujemy naszą usługę w konsoli web w zakładce `profiles`.
Szczegółowy opis profilu znajdziemy w zakładce `Wiki` (można tam przejść klikając `Details`).
Rozejrzyjmy się w szczegółach, zwróćmy uwagę na:


* Parents - profile mogą być dziedziczone, w naszym przypadku automat generujacy profil wykrył, że usługa do działania potrzebuje profili feature-camel i feature-cxf.
* Artifacts - aktualnie mamy jedną paczkę.
* Features - camel-cxf feature.
* Readme - opis z Readme.txt naszej usługi.

## Krok 5 - Uruchomienie własnej usługi ##

    Fabric8 -> Runtime -> Create -> Profiles: add esb/report/service -> Container Type: Child, Container Name: instance01, -> Create and start new container

Przetestuejmy działanie usługi z SoapUI, sprawdźmy koniecznie diagram naszej trasy Camel oraz logi w konsoli web.

Sprawdzenie logów usługi z poziomu konsoli ssh wymaga podłączenia się do kontenera instance01.

    Fabric8:karaf@root> container-connect instance01
    SSH Login for instance01: admin
    SSH Password for admin@instance01:admin
    Fabric8:admin@instance01> log:tail


To już niestety koniec. Powodzenia w hackowaniu Fabric8:)
