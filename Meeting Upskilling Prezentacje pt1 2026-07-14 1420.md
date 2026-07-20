---
categories:
  - "[[Meetings]]"
type:
  - Upskilling
date: 2026-07-14
org:
  - Capgemini
loc:
people: []
topics: []
project:
tags:
  - note
---
## Notes

- nie używać gorna, jest uważany za antipatern( w spolecznosci 50/50), zamiast tego pgx
- OpenAPI do api
- testy nie w katalogach osobnych, powinny byc w tych samych plikach lub pakietach co pliki ktorych dotycza
- testy table driven?![[Pasted image 20260714142306.png]]
- uzywala gin'a, mozna uzywac middleware'a jakiegos
- PRZYPOMNIEC SOBIE CO ZNACZY ***SYGNATURA***
- dobrze jest używać interfejsów! szczególnie na mockach do testów
- migracja? (u niej to tworzenie tabel w gornie)
- **Olek powiedzial ze mozna zrobic certyfikacje industry z retailu!**
- ![[Zrzut ekranu 2026-07-14 140341.png]]


Komentarze po dzisiejszej prezentacji Pauliny. Jako ze pozostali maja wiecej czasu na przygotowanie aplikacji/prezentacji, prosze aby pozostale osoby uwzglednily ponizsze punkty w swoich rozwiazaniach (wszystkie punkty lub chociaz niektore z nich)

- warto umiec pisać testy w stylu table-driven (e.g. [https://go.dev/wiki/TableDrivenTests](https://go.dev/wiki/TableDrivenTests "https://go.dev/wiki/tabledriventests"))
- w projekcie uzywamy JSON Schema do specyfikacji modeli; nastepnie kod golangowy generujemy przy uzyciu [https://quicktype.io/](https://quicktype.io/ "https://quicktype.io/")
- w projekcie uzywamy openapi do specyfikacji API ([https://petstore.swagger.io/#/](https://petstore.swagger.io/#/ "https://petstore.swagger.io/#/"))
- pliki json schema oraz openapi moga byc pozniej uzywane w kodzie do automatycznej walidacji wiadomosci (tzn czy otrzymany request/wiadomosc jest zgodna ze specyfikacja)
- middleware - [https://gin-gonic.com/en/docs/middleware/](https://gin-gonic.com/en/docs/middleware/ "https://gin-gonic.com/en/docs/middleware/") 
- PGX driver (zamiast GORM) - [GitHub - jackc/pgx: PostgreSQL driver and toolkit for Go](https://github.com/jackc/pgx "https://github.com/jackc/pgx")
- warto doczytac o interface'ach oraz mockowaniu
- probujcie robic 'idioto-odporne' aplikacje, tzn. rozne walidacje, obsluga oczywistych bledow (nawet gdy myslicie ze cos sie nie wydarzy, np. nigdy nikt nie bedzie probowac usunac 2 razy ten sam resource)
- logger: w projekcie uzywamy zerolog - dobre logi pomagaja w analizie bledow podczas testow / dzialania na produkcji
- migracjeDB: uzywamy goose
- zrobcie co najmniej jedno flow ktore zawiera w modelu **pola opcjonalne** (requesty powinny przyjmowac nulle/nile a aplikacja powinna je poprawnie obslugiwac), przyklad - 'komentarz do orderu', 
- w wolnym czasie mozecie poczytac - [GitHub - avelino/awesome-go: A curated list of awesome Go frameworks, libraries and software](https://github.com/avelino/awesome-go "https://github.com/avelino/awesome-go")