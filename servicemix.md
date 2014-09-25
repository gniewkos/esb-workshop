Cześć 1 - ServiceMix
====================

### Krok 1 - Generacja usługi z szablonu ####

Rozpoczynamy budowę usługi.

Celem jest zbudowanie usługi proxy z kolejką. 
Aby nie startować od zera rozglądamy się po dostępnych [szablonach startowych Camel](http://camel.apache.org/camel-maven-archetypes.html) (w lokalnym dialekcie archetypes).

Promowany dziś szablon to camel-archetype-cxf-code-first-blueprint. Otwieramy terminal i generujemy startowy projekt.

~~~
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
~~~

Projekt zbudowany. Owieramy go w IntelliJ Idea.

~~~
IntelliJ Idea -> File -> Import Project -> ~/Workspace/esb-workshop/report-service/pom.xml -> 4xNext -> This Window
~~~
