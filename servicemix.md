Cześć 1 - ServiceMix
====================

## Krok 1 - Generacja usługi z szablonu ###

Rozpoczynamy budowę usługi.

Celem jest zbudowanie usługi proxy z kolejką. 
Aby nie startować od zera rozglądamy się po dostępnych [szablonach startowych Camel](http://camel.apache.org/camel-maven-archetypes.html) (w lokalnym dialekcie archetypes).

Promowany dziś szablon to camel-archetype-cxf-code-first-blueprint. Otwieramy terminal i generujemy startowy projekt.


    $ cd ~/Workspace/esb-workshop/
    $ mvn archetype:generate -DarchetypeArtifactId=camel-archetype-cxf-contract-first-blueprint -DarchetypeVersionId=2.13.2 -DarchetypeGroupId=org.apache.camel.archetypes
    ..
    Define value for property 'groupId': : esb
    Define value for property 'artifactId': : report-service
    Define value for property 'version':  1.0-SNAPSHOT: :
    Define value for property 'package':  esb: :
    ..
     Y: :Y
     ..
     [INFO] BUILD SUCCESS

Projekt wygenerowany. Owieramy go w IntelliJ Idea.


    IntelliJ Idea -> Import Project -> ~/Workspace/esb-workshop/report-service/pom.xml -> 4xNext -> This Window


## Krok 2 - wprowadzenie do Apache Camel ##

(Super) Szybkie wprowadzenie do Apache Camel (15 min).

Camel przedstawia się najstępująco:

*Camel (http://camel.apache.org) is an open-source, Java-based project that helps the user implement many of the design patterns in the EIP book.* 

Porozglądajmy się więc po dostępnych [wzorcach](http://camel.apache.org/enterprise-integration-patterns.html).

Możemy potraktować Camela jak klocki Lego. Klocki to [komponenty](http://camel.apache.org/components.html) Camela.
Wzorce to zestawy klocków z instrukcją użycia (klocki możemy składać jak w instrukcji ale nie musimy).
Klocki składamy za pomocą XML lub Javy. Na warsztatach będziemy pracowć z XML.

Największą zaletą składania za pomocą XML jest prostota i możliwość przeczytania pliku `blueprint.xml` (np. przez osoby nie znające Javy, czy narzędzia).
Spróbujemy więc przeczytać co robi wygenerowana usługa.
Można wspomagać się [Camel Getting Started](http://camel.apache.org/book-getting-started.html) lub poniższym skrótem.

Podstawowe pojęcia:

* `Endpoint` może mieć kilka znaczeń zależnie od kontekstu, w analogi Lego jest to złączka. Komponenty udostępniają enpointy w postaci URI np. camel-cxf udostępnia URI: `cxf:<bean:cxfEndpoint|//someAddress>[?options]`.
* `CamelContext` uruchamia wszystkie komponenty wchodzące w skład aplikacji (w naszym przypadku usługi). Może być to aplikacja wielowątkowa - niektóre komponenty uruchamiają własne wątki.
* `Message` komuniakt, np. komunikat rządania klienta, odpowiedzi serwera lub błąd. Message składa się z *id*, *body* oraz *headers*.
* `Exchange` opakowanie dla pojedynczej wymiany komunikatów (żądanie + odpowiedź lub błąd). Exchange może być typu InOut (żądanie odpowiedź) lub InOnly (zdarzenia).
* `Route` trasa jest przejściem `Message` krok po kroku od jednego `Endpointu` do innego.

Podstawowe elementy służące do składania:

* `from` wskazuje na endpoint startowy dla trasy.
* `to` przesyła komunikat do wskazanego endpointu (przez wskazanie URI), odpowiedź z endpointu wstawia do Exchange. 
* `simple` język pozwalający tworzyć proste wyrażenia wyliczeniowe [simple](http://camel.apache.org/simple.html).


    $ read-with-understanding ~/Workspace/report-service/../blueprint.xml

## Krok 3 - Prosta usługa proxy ##

Spróbujmy zmodyfikować usługę (zmieniając blueprint.xml) tak aby otrzymać prostą usługę proxy. 

Usługa proxy niech będzie udostępniona na ServiceMix pod adresem:

    http://localhost:**8888**/report-service/

Reprezentowanym na trasie jako następujący endpoint:

    <cxf:cxfEndpoint id="reportEsbEndpoint"
                     address="http://localhost:8888/report-service/"
                     wsdlURL="wsdl/report_incident.wsdl">
    </cxf:cxfEndpoint>


Ponieważ, nie bedziemy potrzebować zamiany rządania na obiekty Javy nie używamy już atrybuut serviceClass. Endpoint cxf obsługuje formaty: 
    
* POJO (domoślny)
* PAYLOAD(treścią komunikatu jest soap.body)
* RAW(treścią komunikatu jest pełny soap, nie mam możliwości parsowania treści)

W wygnerowanej usłudze używany był domyślny tryb POJO, trzeba go zmienić na PAYLOAD. URI komponentu CXF będzie następujące: 


    uri="cxf:bean:reportEsbEndpoint?dataFormat=PAYLOAD"


Rządanie klienta poiwnno zostać przekazane do usługi docelowej na adresie:

    http://localhost:8088//mock-report-service
    
Reprezentowanym na trasie jako endpoint:

    <cxf:cxfEndpoint id="reportEndpoint"
                     address="http://localhost:8088/mock-report-service/"
                     wsdlURL="wsdl/report_incident.wsdl">
    </cxf:cxfEndpoint>
    
z uri:

    uri="cxf:bean:reportEndpoint?dataFormat=PAYLOAD"
    
Dodatkowo zmieniamy rozszerzenie pliku ReadMe.txt na ReadMe.md (aby ułatwić rozpoznawanie formatu markdown)

Naszym zadaniem jest złożenie trasy `route`.
Wprowadzone modyfikacje instalujemy w lokalnym repozytorium paczek Maven (/home/esb/.m2/repository/).

    $ cd ~/Workspace/esb-workshop/report-service/
    $ mvn install

## Krok 4 - Uruchamiamy ServiceMix ##

    $ cd ~/Programs/apache-servicemix-5.1.2/
    $ ./bin/servicemix

... i rozglądamy się po terenie

Sprawdza się metoda pracy z konsolą linux, działa TAB:)

    karaf@root> h `tab`
    head       headers    help       history
    karaf@root> help

`help` wyświetla listę wszystkich dostępnych poleceń.
Większość z nich udostępnia pomoc `--help`.

Polecenia wystarcząjące do przejścia warsztatu to (także w 90% standardowego użytkowania):

* podpowiedzi `tab`
* wyszukiwanie w historii poleceń `ctrl+r`
* niektóre przydatne linuxowe narzędzia gdzie numer jeden to `grep`
* `list` wyświetla listę paczek (w lokalnym dialekcie *bundli*), najcześciej używana komenda to: `list | grep '..'`
* `log:tail` wyświetla logi ServiceMix
* `install` instaluje nowe paczki
* `uninstall BUNDLE_ID` odinstalowuje paczki
* `start BUNDLE_ID` startuje paczki (działa też `install -s`)
* `stop BUNDLE_ID` zatrzymuje paczki 
* `dev:watch *` automatycznie odświerza paczki rozwojowe (w lokalnym dialekcie *SNAPSHOTS*)

Proponuje poeksperymentować trochę z poleceniami `list` i `log:tail` w połączeniu z `grep`.

    karaf@root> list
    ..
    karaf@root> log:tail
    ...
    ctrl+c
