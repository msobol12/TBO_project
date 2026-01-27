# Proces CI/CD
Proces CI/CD uruchamiany jest automatycznie przy każdej próbie wprowadzenia zmian do repozytorium (push) oraz podczas tworzenia lub aktualizacji żądań typu pull request.
```
on:
  push:
    branches:
      - "**"
  pull_request:
    branches:
      - main
```
## Etapy procesu
Proces CI/CD składa się z kilku etapów
<img width="1350" height="347" alt="image" src="https://github.com/user-attachments/assets/d2b50787-e285-493a-ab98-a952cc0e3fef" />
1. Uruchomienie testów jednostkowych
2. Skan SCA - OWASP
3. Skan SAST - CodeQL
4. Budowa obrazu docer
5. Skan SCA obrazu docker - Trivy
6. Skan DAST - OWASP ZAP
7. Wypchnięcie obrazu docker do rejestru

Jeżeli testy lub któryś ze skanów nie przejdą obraz docker nie jest wypychany, a proces CI/CD kończy się niepowodzeniem.

## Testy jednostkowe
Do uruchomienia testów jednostkowych wykorzystany został pytest
<img width="1097" height="347" alt="image" src="https://github.com/user-attachments/assets/71d41d10-6d90-4fde-89d3-a2cfb53092b1" />

## Skan SCA
Po poprawnym przejściu testów uruchamiana jest statyczna analiza zależności. W tym etapie narzędzie OWASP Dependency-Check skanuje projekt w poszukiwaniu znanych podatności (CVE) w wykorzystywanych bibliotekach. Analiza opiera się na bazie danych NVD, do której dostęp realizowany jest przy użyciu klucza API. W wyniku skanowania generowany jest raport w formacie HTML, który następnie zapisywany jest jako artefakt workflow. Etap ten umożliwia wykrycie potencjalnych podatności w zewnętrznych zależnościach używanych w projekcie.

Raport z narzędzia można znaleźć w jobie https://github.com/msobol12/TBO_project/actions/runs/21392367476 w artefaktach (dependency-check-report)

<img width="1148" height="302" alt="image" src="https://github.com/user-attachments/assets/b9c5b8ae-74ea-46eb-82b9-b2a54bc41488" />

Znaleziono trzy podatności Critical, High, Medium w bibliotekach: form-data, axios, follow-redirects

## Skan SAST
Kolejnym krokiem w procesie CI/CD jest analiza bezpieczeństwa kodu źródłowego. W tym etapie uruchamiane jest narzędzie CodeQL dla języka Python, które wykonuje rozszerzone zapytania bezpieczeństwa z zestawu security-extended. Po zakończeniu analizy sprawdzane są otwarte alerty wygenerowane przez CodeQL, a w przypadku wykrycia jakichkolwiek podatności bezpieczeństwa pipeline kończy się błędem. Etap ten pozwala na identyfikację potencjalnych błędów oraz luk bezpieczeństwa bezpośrednio w kodzie aplikacji.

Raport z narzędzia widoczny jest w zakładce Security -> Code scanning w repozytorium https://github.com/msobol12/TBO_project/security/code-scanning 
<img width="1461" height="822" alt="image" src="https://github.com/user-attachments/assets/8439983d-5281-4e7c-8136-bcbda110e70a" />

Znaleziono 9 podatności (1 High, 8 Medium)

## Skan SCA obrazu docker
Po zbudowaniu obrazu Docker następuje jego analiza pod kątem bezpieczeństwa przy użyciu narzędzia Trivy. Obraz jest skanowany w poszukiwaniu podatności o poziomie LOW, MEDIUM, HIGH oraz CRITICAL, a wyniki skanowania są zapisywane w raporcie w formacie HTML, który następnie staje się artefaktem workflow.

Raport z narzędzia można znaleźć w jobie https://github.com/msobol12/TBO_project/actions/runs/21394161563 w artefaktach (trivy-report). W tym jobie tymczasowo zmieniono zasady wykonywania procesu CI/CD – usunięto wymóg przeprowadzania testów SCA i SAST przed zbudowaniem obrazu Docker. W przeciwnym wypadku raport z Trivy nie powstałby, ponieważ skany SCA i SAST generują alerty, które kończą proces przed etapem skanowania obrazu.
<img width="1213" height="539" alt="image" src="https://github.com/user-attachments/assets/f47f494d-860c-4e56-a5b5-1bda9c617e32" />

## Skan DAST
Po skanowaniu obrazu uruchamiany jest kontener Docker z aplikacją, a następnie wykonywany jest OWASP ZAP Baseline Scan na działającej aplikacji. W przypadku wykrycia podatności pipeline zostaje przerwany, a raport z testów jest dostępny bezpośrednio w jobie. Etap ten pozwala na identyfikację podatności, które są widoczne z perspektywy użytkownika aplikacji.

Wynik wykonania można zobaczyć bezpośrednio w jobie https://github.com/msobol12/TBO_project/actions/runs/21392125756/job/61581370783 . W tym jobie tymczasowo zmieniono zasady wykonywania procesu CI/CD – usunięto wymóg przeprowadzania testów SCA i SAST przed zbudowaniem obrazu Docker. W przeciwnym wypadku skan DAST nie uruchomił by się, ponieważ skany SCA i SAST generują alerty, które kończą proces przed etapem skanowania obrazu.

<img width="949" height="146" alt="image" src="https://github.com/user-attachments/assets/0493c03e-d258-40a4-bdb6-fe981d635488" />

<img width="895" height="528" alt="image" src="https://github.com/user-attachments/assets/66228187-b173-471b-8808-d4323bd5eaa0" />

<img width="869" height="657" alt="image" src="https://github.com/user-attachments/assets/4296c260-7138-4b48-add8-4d2fd09c0650" />

<img width="805" height="323" alt="image" src="https://github.com/user-attachments/assets/f4476866-50ce-48ed-8d96-338f43f30372" />

Znaleziono 12 warningów

## Obraz docker
Obraz dla gałęzi main jest budowany z tagiem latest, dla wszystkich pozostałych gałęzi ma tag beta.

Po zbudowaniu obrazu i przejściu wszystkich testów bezpieczeństwa, obraz jest publikowany w Docker Hub https://hub.docker.com/r/tboproject1/tbo-app/tags






