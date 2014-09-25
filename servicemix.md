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

Polecenia wystarczające do przejścia warsztatu to (także w 90% standardowego użytkowania):

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
    
## Krok 5 - Uruchamiamy usługę ##

Uruchamiamy usługę prostego proxy na ServiceMix.

ServiceMix jest domyślnie skonfigurowany w taki sposób, aby szukać paczek właśnie w `~/.m2/repository/`. Dzięki temu instalacja paczki na ServiceMix polega na wykonaniu polecenia.

    karaf@root> install -s mvn:esb/report-service/1.0-SNAPSHOT
    Bundle ID: 200

Sprawdzamy status usługi. `Active` = OK. 

    karaf@root> list
    ...
    20   Active  Created   80    A Camel CXF Blueprint Route (1.0.0.SNAPSHOT)

Sprawdzamy logi z uruchomienia usługi.

    karaf@root> log:tail
    ...
    ctrl+c

ServiceMix udostępnia (z wykorzystaniem biblioteki cxf) listę dostępnych usług po adresem:
    
    http://localhost:8181/cxf

Możemy wyświetlić wsdl usługi dodając parametr ?wsdl do adresu:
    
    http://localhost:8888/report-service/?wsdl


Ustawiamy ServiceMix aby przyszłe zmiany usługi były odświerzane automatycznie po zainstalowaniu w lokalnym repozytorium (zmiany wszystkich paczek SNAPSHOTS).

    karaf@root> dev:watch *

Aby przetestować odświerzanie modyfikujemy nazwę usługi w pliku pom.xml

    <name>Report Service</name>
    
Przebudowujemy usługę i sprawdzamy konsolę ServiceMix (polecenie `list` wyświetli nową nazwę usługi).

    $ cd ~/Workspace/esb-workshop/report-service/
    $ mvn install
    
## Krok 6 - Testy SaopUI ##

Testujemy usługę prostego proxy z SoapUI.

Uruchamiamy SoapUI i na podstawie WSDL generujemy żądanie.

    SoapUI -> File -> New SOAP Project -> Project Name: report-service, Initial WSDL: http://localhost:8888/report-service/?wsdl -> OK
    
Generujemy mock usługi docelewej.

    SoapUI -> ReportIncidentBinding -> Generate SOAP Mock Service -> Path: /mock-report-service, Port: 8088 -> 2xOK


Mock zostaje wygenerowany, starujemy go przez kliknięcie zielonej strzałki (okno mocka otwiera się automatycznie lub po klinięciu nazwy mocka).
Wszystko gotowe, możemy wysłać rządanie na adres usługi na ServiceMix. 


Wysyłamy żądanie na adres usługi : hhttp://localhost:8888/report-service/

    <soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:rep="http://reportincident.example.camel.apache.org">
       <soapenv:Header/>
       <soapenv:Body>
          <rep:inputReportIncident>
             <incidentId>1</incidentId>
             <incidentDate>2014-09-19</incidentDate>
             <givenName>Man Who</givenName>
             <familyName>Loves Simplicity</familyName>
             <summary>I am despair</summary>
             <details>Nothing is as easy as it could...</details>
             <email>man_who_loves_simplicity@pl.sii.eu</email>
             <phone>777 777 777</phone>
          </rep:inputReportIncident>
       </soapenv:Body>
    </soapenv:Envelope>

W dolnej częsci okna mocka wyświetlana jest list z logiem `Message Log`. Powinien pojawić się nowy wpis, który możemy podejrzeć przez kliknięcie.
    
## Krok 7 - Logowanie ##

Logowanie komunikacji.

Sprawdzamy logi z przepływu komunikatu na ServiceMix.

    karaf@root> log:tail
    ...
    ctrl+c

Log trafia także do pliku:

    tail ~/Programs/apache-servicemix-5.1.2/data/log/servicemix.log
    
Domyślnie nic nie zostało zalogowane. Musimy samodzielnie dodać logowanie.
Logowanie możemy wykonać przy użyciu endpointu log (loggingCategory to dowolny tekst opisujący wpis logu):

    <to uri="log:loggingCategory"/>

Sprawdźmy działanie takiego logowania.

Zagadka - jakich przypadków ten sposób nie zaloguje?

## Krok 8 - Ulepszenie logowania ##

Wykorzystamy logowanie udostępnione przez komponent camle-cxf. 
Poza camelContext definiujemy logowanie jako:

     <bean id="loggingFeature" class="org.apache.cxf.feature.LoggingFeature">
            <property name="prettyLogging" value="true"/>
     </bean>

Aby właczyć tak zdefiniowane logowanie w endpoint:

    <cxf:cxfEndpoint id="reportEsbEndpoint"
                     address="http://localhost:8888/report-proxy/"
                     wsdlURL="wsdl/report_incident.wsdl">
        <cxf:features>
            <ref component-id="loggingFeature"/>
        </cxf:features>
    </cxf:cxfEndpoint>    

Poprzednie logowanie możemy już usunąć.

Przebudowujemy uslugę i ....

    Error updating bundle.
    org.osgi.framework.BundleException: Unresolved constraint in bundle report-service [200]: 
    Unable to resolve 204.3: missing requirement [200.3] osgi.wiring.package; (&(osgi.wiring.package=org.apache.cxf.feature)(version>=3.0.0)(!(version>=4.0.0)))
    
Nie udało się rozwiązać zależności do paczki cxf z wersją 3.0.0 lub większą. 
Sprawdzając pom.xml widzimy, że pomino wskazania wersji szablonu 2.13.2 (-DarchetypeVersionId=2.13.2) wygenerowane wersje zależności to camel 2.14.0 i cxf 3.0.1.

Niestety musimy zmienić wersje na camel **2.13.2** i cxf **2.7.11** (Ctrl+R w IntelliJ).

Przebudowujemy usługę i testujemy w SoapUI nowe logowanie.

## Krok 9 - wstawienie komunikatu na kolejkę ##

Wstawienie komunikatu na kolejkę.

Instalujemy na ServiceMix system obsługi kolejek ActiveMQ (w lokalnym dialekcie Broker komunikatów).
Będzie potrzebny także komponent Freemarker.

    karaf@root> features:install activemq-broker
    karaf@root> features:install camel-freemarker

Komunikat możemy wysłać na kolejkę następująco (inOnly - wysyłamy ale nie potrzebujemy odpowiedzi od Brokera).

    <inOnly uri="activemq://inputReportIncident"/>

Komunikat będzie dostarczony przez szynę offline więc musimy wygenerować komunikat odpowiedzi i przesłać go klientowi.
Do tego rodzaju operacji bardzo czesto wykorzystywany jest Freemarker uzupełniający podany szablon danymi. W naszym przypadku szablonem jest plik ok.xml.

    <to uri="freemarker:ok.xml"/>

Całość trasy wstawiającej komunikat na kolejkę i wysyłającej odpowiedź klientowi wygląda następująco.

    <route>
      <from uri="cxf:bean:reportEsbEndpoint?dataFormat=PAYLOAD"/>
      <inOnly uri="activemq://inputReportIncident"/>
      <to uri="freemarker:ok.xml"/>
    </route>

Możemy przebudować usługę z taką trasą i sprawdzić działanie w SoapUI.
Klient wysyłający rządanie dostaje odpowiedź ale w mocku usługi docelowej nie mamy nowego wpisu.

Nasz komunikat cały czas czeka na kolejce ActiveMQ:

    http://localhost:8181/activemqweb/
   
Zadanie do wykonania to dodanie drugiej trasy pobierającej komunikaty z kolejki (from) i przesyłającej klientowi (to).

Przebudowujemy usługę i sprawdzamy powtórnie kolejkę ActiveMQ. 
    
## Krok 10 - obsługa błędów ##

Dodadamy ponawianie przy wystąpieniu błędów. 

Wymagania obsługi błędów definiowane są często w następujący sposób. 

* Jeśli jest szasna, że błąd jest tymczasowy to usługa powinna ponawiać wysyłanie komuniaktu. 
* Jeśli zostanie osiągnięta maksymalna ilość ponowień to komunikat powinien trafić na "bocznicę".
* Jeśli wiadomo, że błąd samoczynnie nie ustąpi to komunikat powinien od razu trafić na "bocznicę".

Wymagania te ma szanse spełnić jeden z dostępnych w Camel wzorców - [Dead Letter Channel](http://camel.apache.org/dead-letter-channel.html).

    <bean id="myDeadLetterErrorHandler" class="org.apache.camel.builder.DeadLetterChannelBuilder">
        <property name="deadLetterUri" value="activemq://ActiveMQ.DLQ" />
        <property name="redeliveryPolicy">
            <bean class="org.apache.camel.processor.RedeliveryPolicy">
                <property name="maximumRedeliveries" value="5" />
                <property name="redeliveryDelay" value="5000"/>
                <property name="useExponentialBackOff" value="false" />
            </bean>
        </property>
    </bean>

Podłączenie handlera do CamelContext (czyli dla wszystkich tras w CamelContext).

    <camelContext xmlns="http://camel.apache.org/schema/blueprint" errorHandlerRef="myDeadLetterErrorHandler">

Aby zastosować obsługę tylko dla wybranych rodzajów błędów należy użyć klauzule onException (może być ich kilka w CamelContex ale muszą być zdefiniowane jako pierwsze).

    <onException >
        <exception>java.lang.Exception</exception>
        <redeliveryPolicy maximumRedeliveries="10"
                          useExponentialBackOff="true"/>
        <to uri="activemq://ActiveMQ.DLQ"/>
    </onException>

Spróbujmy podłączyć handler globalny i przetestować działanie za pomocą SoapUI (np.wyłączając mocka usługi docelowej).

Zagadka: jakie mogą być problemy z powyższym rozwiązaniem?      