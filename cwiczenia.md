# CWICZENIA

## Pierwsza konfiguracja

- Vagrant - dwie maszyny (jenkins, node), instalacja Jenkinsa
`vagrant up`

Otwórz w przeglądarce adres http://10.0.0.10:8080 i odblokuj Jenkinsa zgodnie z wyświetloną instrukcją.
Na ekranie „Dostosuj Jenkinsa” wybierz opcję „Zainstaluj sugerowane wtyczki”.
Utwórz konto administratora
Login: admin
Hasło: Infoshare%%14.05
Nazwa: admin
Email: admin@local
nowa linia
new2

## Instalacja plugina, zmiana języka
Zmiana języka interfejsu Jenkinsa możliwa jest poprzez plugin. W celu instalacji pluginu przejdź do Zarządzanie Jenkinsem > Zarządzaj wtyczkami > Dostępne i wyszukaj wtyczkę o nazwie Locale.
Zapoznaj się z opisem wtyczki klikając w nazwę wngrokyszukanej pozycji.
Następnie wróć do karty z Jenkinsem, zaznacz checkbox i rozpocznij instalację pluginu poprzez przycisk Install without restart.

Otwórz Zarządzanie Jenkinsem > Skonfiguruj system. Wyszukaj sekcję Locale, w polu Default Language wprowadź wartość EN, zaznacz poniższy checkbox i zapisz.

## 01 - Pierwszy projekt
Cel: Zapoznanie z interfejsem, artefaktami, logami

Ze strony głównej wybierz „New Item”. Podaj nazwę projektu: first-freestyle
Wybierz „Freestyle project” i potwierdź poprzez przycisk OK

Przejdź na dół do sekcji Build -> Add build step -> wybierz Execute shell

W polu command wpisz:
```
date
FILE=$(date +"%S-%M-%H-%m-%d-%y")
hostname > $FILE.txt
echo $JOB_NAME >> $FILE.txt
printenv >> $FILE.txt
```

Następnie z sekcji „Post-build Actions” wybierz Add post-build actions -> Archive the artifacts
W polu wpisz wartość *.txt
Zapisz zmiany i uruchom zadanie.
Po kilku sekundach uruchom zadanie ponownie. Wybierz ostatni numer zadania z listy "Build history" i przejdź do artefaktów. Zajrzyj także do logów konsoli
Kliknij przycisk Configure, w sekcji "Build environment" zaznacz opcję Delete worspace before build starts.
Uruchom ponownie zadanie, a następnie sprawdź różnicę w artefaktach i logach.

## 02 - Integracja z Githubem
Cel: build jako reakcja na zmiany w repozytorium
Potrzebne będzie wystawienie naszego localhosta na świat i dodanie webhooka w Github

Utwórz konto na stronie https://dashboard.ngrok.com/signup. Możliwe jest zalogowanie kontem github. Po zalogowaniu widoczna jest instrukcja instalacji.
W skrócie "Download for Linux" -> `tar -xf <ngrok-file.tgz>` -> Add ngrok to PATH

Jak masz już przygotowany program ngrok u siebie na maszynie, to wymagane jest jedynie uruchomienie komendy w terminalu hosta `ngrok config add-authtoken <token>` z punktu 2 instrukcji
Następnie wykonaj:
`vagrant share jenkins` 
W przypadku problemów z powyższą komendą lub z pluginem vagrant-share zdefiniowanym w Vagrantfile, można manualnie uruchomić ngrok.
`ngrok http 8081`
ngrok wyświetli publiczny adres URL działającego na localhost:8081 Jenkinsa

Wykonaj fork repozytorium i utwórz branch github-integration

W Github rzejdź do Settings > Webhooks > Add webhook i w polu Payload URL podaj adres wygenerowany przez komendę vagrant/ngrok i doklejając na końcu „/github-webhook/”. Np.: https://8fe3-46-204-72-20.eu.ngrok.io/github-webhook/
Następnie kliknij Add webhook. Do poprawnego zadziałania webhooka repozytorium musi być publiczne (rekomendowane na potrzeby wasztatów)
Jeśli chcesz skonfigurować integrację z repozytorium prywatnymm najlepiej jest utworzyć Github App w Githubie i dodatkowe credentials w Jenkins. Dokładna instrukcja pod tym linkiem:
https://github.com/jenkinsci/github-branch-source-plugin/blob/master/docs/github-app.adoc
https://docs.github.com/en/developers/apps/building-github-apps/creating-a-github-app

Jeśli udało się przejść powyższe kroki, a webhook nie działa poprawnie, może być to spowodowane działaniem programu antywirusowego lub konfiguracją sieci.
W takiej sytuacji warto sprawdzić program antywirusowy lub przetestować przy pomocy innego łącza, np. hotspot z telefonu.

Dalej w Manage Jenkins > Global Tool Configuration w sekcji Maven dodaj instalację (maven-apache-3.8.4) i zapisz zmiany


Utwórz Freestyle project.
W SCM > Git podaj adres własnej kopii repo
Branch: */github-integration
Additional Behaviours -> Polling ignores commits in certain paths -> Included Regions: java-app/.*

Build triggers: GitHub hook trigger for GITScm polling
Dodaj następujące zadania budowania:
- Invoke top-level Maven targets:
    - Version: maven-apache-3.8.4
    - Goal: package
    - Advanced -> POM -> wskaż całą ścieżkę do pliku pom e.g. java-app/pom.xml
- Execute shell
    - Command: java -jar java-app/target/my-app-1.0-SNAPSHOT.jar
Dodaj Post-build Actions:
- Publish JUnit test result report: **/surefire-reports/TEST-*.xml
- Archive the artifacts: java-app/target/*.jar
Wykonaj zmianę na branch’u github-integration. Kilka sekund później powinien zostać uruchomiony automatycznie nowy build, który zbuduje, puści testy, uruchomi aplikację i opublikuje artefakty.

trigger1
trigger2
trigger3





## 03 - Dodawanie node'a
Cel: chcemy, aby joby nie były puszczane na wbudowanym, domyślnym nodzie, tylko na dedykowanym, przez nas skonfigurowanym nodzie

Połącz się z maszyną node (vagrant ssh node).
Zainstaluj Jave, natępnie utwórz katalog domowy, użytkownika `jenkins` Nadaj mu odpowiednie uprawnienia. Utwórz klucz rsa i następnie dodaj go do authorized keys. Będzie on używany do połączenia node'a i jenkinsa.

```
sudo apt install openjdk-11-jre
sudo mkdir /var/lib/jenkins
sudo useradd -d /var/lib/jenkins jenkins
sudo chown -R jenkins:jenkins /var/lib/jenkins
sudo su - jenkins
mkdir /var/lib/jenkins/.ssh
ssh-keygen
cat ~/.ssh/id_rsa.pub > /var/lib/jenkins/.ssh/authorized_keys
cat ~/.ssh/id_rsa
rm ~/.ssh/id_rsa
```

W GUI Jenkinsa pójdź do Manage Jenkins -> Manage Nodes and Clouds -> New Node
Node name: node01
Remote root repository: /var/lib/jenkins
Labels: node
Usage: Only build jobs with label expressions matching this node
Launch method: Launch agent via SSH
Host: 10.0.0.20

W sekcji credentials kliknij Add > SSH Username with private key, podaj nazwę użytkownika (jenkins) oraz utworzony poprzednio klucz prywatny.
Host key verification strategy: Manually trusted key verification strategy
Zaznacz checkbox: Require manual verification of initial connection

Zapisz ustawienia i w Trust SSH Host Key zatwierdź klucz hosta.

Po zatwierdzeniu sprawdź status node’a - powinien być online.
Przejdź do ostatniego projektu i wymuś uruchomienie tylko na nowym węźle podając jego label. (Restrict where this project can be run)

Uruchom budowanie i sprawdź logi, czy faktycznie wykonał się na skonfigurowanym nodzie.

## 04 - Pipeline-as-a-code przykład
Cel: Utworzenie pipeline'u w wersji pipeline-as-a-code, Jenkinsfile
https://www.jenkins.io/doc/book/pipeline/jenkinsfile/

W Twoim repozytorium utwórz branch: pipeline-as-a-code

Utwórz projekt typu Pipeline

Build triggers: GitHub hook trigger for GITScm polling

Pipeline script from SCM -> Script path Jenkinsfile-pipeline-2 lub Jenkinsfile-pipeline
Plik zawiera te same kroki, które robiliśmy w ćwiczeniu 02 tylko w postaci Jenkinsfile

## 05 - Pipeline-as-a-code Pierwszy projekt
Cel: Utworzenie Pipeline'u, korzystając z deklaratywnego kodu, bazując na ćwiczeniu 01 z dodatkowymi modyfikacjami (blokowanie kolejnego kroku, reakcje na zdarzenia)

Utwórz Pipeline który:
- Uruchomi się na konkretnym agencie
- Wyświetla timestamp w logach konsoli
- Zainstaluje maven w konkretnej wersji
- Wyczyści workspace od razu po uruchomieniu
- W stage build utworzy plik z aktualną datą z rozszerzeniem txt, a następnie w razie
powodzenia zarchiwizuje plik
- W stage validation zapyta użytkownika czy kontynuować
- W stage deploy wyświetli wersję javy, a następnie nodejs, a w przypadku niepowodzenia zwróci komunikat
„error!”

## 06 - Java Docker
Cel: Uruchomienie aplikacji na agencie w kontenerze

Konfiguracja:
Zainstaluj dockera na maszynie node i dodaj użytkownika jenkins do grupy docker: `sudo usermod -aG docker jenkins` (Można np. użyć części skryptu provision.sh lub pliku provision-node-docker.sh, dodać krok do pliku Vagrant i vagrant reload --provision)
Zainstaluj 2 pluginy w Jenkinsie: Docker plugin i Docker.
Zrestartuj maszyny node i jenkinsa

Uruchomienie w kontenerze
Na podstawie poprzednich ćwiczeń stwórz pipeline dla aplikacji Java, który:
- Uruchomi pipeline w kontenerze (przykład: image 'maven:3.6.1-alpine')
- Będzie zawierał stage: Build, Test, Deliver, Run, gdzie odpowiednio zbuduje paczkę, uruchomi testy i opublikuje ich rezultat, opublikuje artefakt i uruchomi aplikację

## 08 - Python
Cel: Uruchomienie aplikacji python w kontenerze na agencie

Wykonaj pipeline który uruchomi kontener z pythonem na maszynie node.
Folder w repo python-app

W kontenerze muszą zostać zainstalowane zależności, a następnie aplikacja musi być uruchomiona.
W kolejnym etapie (test) zweryfikuj działanie aplikacji komendą wget.

## 09 - Python - budowanie obrazów docker
Cel: Budowanie obrazów Docker z aplikacją w pipelinie i publikowanie jako artefakt

Wykonaj pipeline, który zbuduje obraz dockera na bazie Dockerfile.
Następnie opublikuje plik obrazu jako artefakt pipeline i wyczyści workspace

## 10 - Python - integracja z Dockerhub
Cel: Zbudowanie obrazu Docker i opublikowanie do Dockerhub

Wykonaj pipeline, który zbuduje obraz dockera na bazie Dockerfile.
W Jenkins skonfiguruj credentials, które użyjesz do zalogowania do Dockerhub w pipelinie
Następnie wrzuć zbudowany obraz do Dockerhub

