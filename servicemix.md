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


## Krok 2 ##

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
