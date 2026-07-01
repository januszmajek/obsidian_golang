---
tags:
  - cv
  - "#cap"
---
# Moja historia z GO
1. Zajawiłem sie go od kolegi ze studiów z którym często rozmawiam o programowaniu
   
2. już kiedy kończył się mój pierwszy projekt interesowałem się na technologie backendowe jakich chce sie nauuczyc zeby poszerzyc kompetencje i GO jest jednym z nich, pytałem też Tomka czy mógłby znaleźć dla mnie jakiś projekt
   
3. w trakcie mojego czasu na benchu pomagałem też w projekcie wewnętrznym dla studentów, tam zajmowałem sie javą, w trakcie miałem rzeczy związane z kupnem i wykończeniem mieszkania które ciągną sie do teraz przez co nie mialem czasu na GO
   
4. po MyPlace i CargoMate stwierdzam ze java i jej **drzewo zależności**, ilością **boilerplate kodu**, obsługiwaniem wyjątków mi nie leżą.
O mnie
- szybko się ucze nowych technologii, nie mam problemu z eksperymentowaniem, chciałbym sie określić w kierunku GO na backendzie
- NIE PROGRAMOWALEM NIC SAM, moje zainteresowania ograniczaly sie do ogladania YT, discord programistyczny
- glownie frontend dev, z checia przejscia na fullstackowość z GO


# GO WIEDZA
- asynchroniczność w GO
- obiektowośc w GO, nie ma dziedziczenia, jest inna niz w javie, do struktur danych dopisujesz metody, obiektowośc jako **enkapsulacja**
- GO rutyny, (rutyna to jest wątek, w for mozna odpalic 1k wątków), współbieżność
- GO structs - what is it, use cases, examples
- mutex
- indexowanie w bazach danych
- ogarnac mikroserwisy
- wzorce projektowe (builder pattern, decorator pattern, singleton, pattern, fasade pattern, observer pattern, **wzorzec strategii** i tym podobne)
# CHAT GO CLAUDE
## Asynchroniczność w GO (Asynchronous Programming)
Go wykorzystuje model współbieżności oparty na komunikacji sekwencyjnych procesów (CSP - Communicating Sequential Processes). Asynchroniczność realizowana jest poprzez gorutyny (goroutines) i kanały (channels). Kanały służą do bezpiecznej komunikacji między gorutynami, eliminując potrzebę jawnego zarządzania blokadami. Przykład: go funkcja() uruchamia funkcję asynchronicznie, a ch := make(chan int) tworzy kanał do przesyłania danych typu int.

## Obiektowość w GO (Object-Oriented Programming)
Go nie implementuje klasycznej obiektowości znanej z Javy. Nie ma klas ani dziedziczenia (inheritance). Zamiast tego wykorzystuje struktury (structs) i interfejsy (interfaces). Metody przypisuje się do typów poprzez odbiorniki (receivers). Enkapsulacja realizowana jest przez konwencje nazewnictwa - wielkie litery oznaczają publiczny dostęp, małe - prywatny. Przykład: func (p Person) GetName() string { return p.name } definiuje metodę dla struktury Person.

## Gorutyny (Goroutines)
Gorutyny to lekkie wątki zarządzane przez środowisko uruchomieniowe Go. Można utworzyć tysiące gorutyn bez znaczącego obciążenia systemu. Każda gorutyna zajmuje początkowo około 2KB pamięci. Uruchamiane są słowem kluczowym go. W pętli for można łatwo uruchomić tysiąc gorutyn: for i := 0; i < 1000; i++ { go worker(i) }. Go scheduler automatycznie zarządza ich wykonaniem na dostępnych rdzeniach procesora.

## Struktury (Structs)
Struktury w Go to typy danych grupujące powiązane dane. Definiuje się je słowem kluczowym type. Przykład: type User struct { Name string; Age int }. Struktury mogą mieć metody, być zagnieżdżane i implementować interfejsy. Często używane do modelowania encji biznesowych, konfiguracji aplikacji czy reprezentacji danych JSON. Inicjalizacja: user := User{Name: "Jan", Age: 30} lub user := User{} z późniejszym przypisaniem wartości.
## Mutex (Mutual Exclusion)
Mutex to mechanizm synchronizacji zapobiegający jednoczesnej modyfikacji współdzielonych zasobów przez wiele gorutyn. Go oferuje sync.Mutex dla podstawowej blokady i sync.RWMutex dla odczytu-zapisu. Użycie: mu.Lock() blokuje dostęp, mu.Unlock() odblakowuje. RWMutex pozwala na wielokrotny równoczesny odczyt, ale wyłączny zapis. Kluczowe dla zapobiegania wyścigom danych (race conditions) w aplikacjach współbieżnych.
Indeksowanie w bazach danych (Database Indexing)
Indeksy w bazach danych to struktury danych przyspieszające wyszukiwanie. Najczęściej używane typy to B-tree dla równości i zakresów, hash dla równości, bitmap dla danych kategorycznych. Indeksy poprawiają wydajność SELECT, ale spowalniają INSERT/UPDATE/DELETE. W Go przy pracy z bazami używa się bibliotek jak database/sql lub ORM jak GORM. Przykład definicji w SQL: CREATE INDEX idx_user_email ON users(email).
## Mikroserwisy (Microservices)
Mikroserwisy to architektura dzieląca aplikację na małe, niezależne usługi komunikujące się przez API. Go doskonale nadaje się do mikroserwisów dzięki szybkiemu startowi, małemu zużyciu pamięci i wbudowanemu serwerowi HTTP. Kluczowe aspekty: niezależne wdrażanie, oddzielne bazy danych, komunikacja przez REST/gRPC, zarządzanie przez Docker/Kubernetes. Wyzwania to zarządzanie konfiguracją, monitorowanie i obsługa błędów rozproszonego systemu.
## Wzorce projektowe (Design Patterns)
**Builder Pattern**: Konstrukcja złożonych obiektów krok po kroku. W Go: type UserBuilder struct{} z metodami SetName(), SetAge(), Build().
**Decorator Pattern**: Dodawanie funkcjonalności bez modyfikacji oryginalnego kodu. Implementacja przez interfejsy i kompozycję.
**Singleton Pattern**: Jeden egzemplarz klasy w całej aplikacji. W Go przez sync.Once: var instance *Singleton; once.Do(func() { instance = &Singleton{} }).
**Facade Pattern**: Uproszczony interfejs do złożonego systemu. Struktura łącząca wiele mniejszych komponentów.
**Observer Pattern**: Powiadamianie o zmianach stanu. Implementacja przez interfejsy i slice obserwatorów.
**Strategy Pattern**: Wybór algorytmu w czasie wykonania. Interfejs definiujący strategię i różne implementacje.
	- 
# W CM
- JAVA 17, JMS, SPRIG BOOT
- ANGULAR 18, TS, MATERIALUI, BOOTSTRAP, MODULE FEDERATION
	- ### 🔧 Jak to działa?

1. **Host i Remote**:
    
    - **Host**: Aplikacja główna, która ładuje inne aplikacje (remote).
    - **Remote**: Aplikacja, która udostępnia swoje moduły lub komponenty do załadowania przez hosta.
2. **Webpack Module Federation Plugin**:
    
    - Konfiguracja odbywa się w pliku `webpack.config.js` lub `webpack.config.ts` (jeśli używasz Angular CLI z `@angular-architects/module-federation`).
    - Określasz, które moduły mają być eksportowane (`exposes`) i które mają być importowane (`remotes`).
3. **Dynamiczne ładowanie**:
    
    - Angular ładuje zdalne moduły w czasie rzeczywistym, np. z innego serwera lub aplikacji.
    - Dzięki temu możesz aktualizować jedną część systemu bez potrzeby przebudowy całego monolitu.
    ### Zalety

- Niezależne wdrażanie aplikacji.
- Skalowalność zespołów i kodu.
- Szybszy czas ładowania dzięki dzieleniu kodu.

### Wyzwania

- Zarządzanie zależnościami (np. wersje Angulara muszą być zgodne).
- Potencjalne problemy z wydajnością i bezpieczeństwem przy złej konfiguracji.



# po rozmowie
na pewno powiedzą nie, znałem pół odpowiedzi z kilku pytań
- nie wiedzialem czym jest kolejkowość
- nie umiałem odpowiedziec czemu nie ma dziedziczenia ale jest ono możliwe do zrealizowania
- nie wiedzialem czym jest view w tabeli a sama tabela
- nie znalem zadnego wzorca projektowego 
	- powiedzialem ze glownie mapujemy dane z jednej aplikacji do drugiej i ciagnal zebym powiedzial pod jaki wzorzec to podchodzi a ja nie wiedzialem, jaki to wzorzec
- **nie jestem pewien czy udzieliłem dobrej odpowiedzi o e2e integracji i unitach**
- pytali czy pracowałem w kolejkach na backendzie
- pytali czy pracowałem z mikroserwisami, odpowiedzialem ze w CM mamy mikroserwisy
- skoro mówiłem że nic nie pisałem a kojarzę nowinki to spytali jaka nowinkę ostatnio słyszałem 
- pytanie o indeksy w bazie danych
- pytanie GO rutyny
---
# moje wrażenia po rozmowie
- byłem bardzo chaotyczny, kiedy mówiłem o czymś, o czym **wydaje mi się** że coś wiem (e2e)
- było pojedyncze **wiem**, prawie wszystkie inne pytania były **nie wiem**
- biorąc pod uwagę że pytali Krzyśka godziny wcześniej to, uważam, że jeżeli Tomek jakoś nie zaczaruje w tle, to nie mam szans sie tam po tej rozmowie dostac
- powinienem zadawać pytania do ich pytania, na przyklad wymien jakikolwiek znasz wzorzec to o jakies konkretne pytanie prosze, bo to bardzo ogolne pytanie, w mojej praktyce malo kto sie zastanawia 
- **nie znam pierdolonych podstaw**