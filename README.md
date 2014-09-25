esb-workshop
============

From ServiceMix to Fabric8

## Cel warsztatu ##

* Zapoznanie z frameworkiem integracyjnym [Apache Camel](http://camel.apache.org/)
* Zapoznanie z lekkim rozwiązaniem esb [ServiceMix](http://servicemix.apache.org/)
* Zapoznanie z kierunkami rozwoju narzędzi integracyjnych [Fabric8](http://fabric8.io/)
* Wyrobienie sobie opinii o możliwosciach zastosowania

## Uruchomienie dostarczonego obrazu Linux Mint 17 ##

Obraz zawiera:
 
 * Zainstalowana Java 1.7.0_67, Apache Maven 3.2.1, Git 1.9.1
 * Zainstalowany SaopUI 5.0.0 oraz IntelliJ Idea 13.1.4
 * Rozpakowaną w folderze ~/Programs dysrybucję ServiceMix 5.1.2 oraz Fabric8 1.2.0.Beta4.

Działanie obrazu jest sprawdzone pod VirtualBox 4.3.16 (wiadomo o problemie ze zwiększonym zużyciem procesora pod jedną z wcześniejszych wersji).

## Warsztat składa się z dwóch części ##

 * Cześć 1 - ServiceMix (esb-workshop/servicemix.md)
 * Cześć 2 - Fabric8 (esb-workshop/fabric8.md)

## Praca z repo ##

Klonujemy repo 
    
    $ cd ~/Workspace
    $ git clone https://github.com/gniewkos/esb-workshop.git

Warsztat podzielony jest na 10 kroków budowy usługi i uruchamiania na ServiceMix oraz ostatni krok uruchamiania na Fabric8.
Każdy krok jest otagowany.

    $ cd esb-workshop
    $ git tag
    fabric8
    step1
    step10
    step2
    step3
    step4
    step5
    step6
    step7
    step8
    step9

Rozpoczynamy od kroku 1

    git checkout tags/step1

Aby przeskoczyć do jakiegoś kroku wycofując własne eksperymenty wykonujemy:

    $ git checkout .
    $ git checkout tags/step


    
    