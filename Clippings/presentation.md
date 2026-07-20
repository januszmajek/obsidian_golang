# mini-wms — Prezentacja projektu

## Czym jest ten projekt?

**mini-wms** to uproszczony system zarządzania magazynem (ang. _Warehouse Management System_) napisany w języku **Go**. Projekt powstał jako ćwiczenie praktyczne w ramach nauki Go i ma na celu odwzorowanie podstawowych procesów magazynowych:

- rejestrowanie produktów,
- przyjmowanie towaru na magazyn (operacje przychodzące, _inbound_),
- tworzenie zamówień i ich realizacja (_shipping_),
- podgląd aktualnego stanu magazynowego.

Aplikacja udostępnia REST API oparte na frameworku **Gin**, a dane przechowuje w bazie **PostgreSQL**.

---

## Struktura projektu

```
wms_golang/
├── cmd/api/          # punkt wejścia aplikacji
├── internal/
│   ├── config/       # konfiguracja
│   ├── db/           # połączenie z bazą danych
│   ├── product/      # moduł produktów
│   ├── stock/        # moduł stanu magazynowego
│   └── order/        # moduł zamówień
├── migrations/       # migracje bazy danych (goose)
├── docker-compose.yml
├── go.mod / go.sum
└── presentation.md
```

---

## Opis plików

### `cmd/api/main.go`
Główny plik startowy aplikacji. Odpowiada za:
- załadowanie konfiguracji (`config.Load()`),
- nawiązanie połączenia z bazą danych,
- inicjalizację struktur `*DB` dla każdego modułu,
- zbudowanie routera Gin i uruchomienie serwera HTTP.

Zawiera również funkcję `newRouter`, która rejestruje wszystkie endpointy REST:

| Metoda | Ścieżka             | Opis                          |
|--------|---------------------|-------------------------------|
| GET    | `/health`           | Health check                  |
| GET    | `/products`         | Lista produktów               |
| POST   | `/products`         | Dodaj produkt                 |
| POST   | `/inbounds`         | Przyjmij towar na magazyn     |
| GET    | `/stock`            | Raport stanu magazynowego     |
| POST   | `/orders`           | Utwórz zamówienie             |
| GET    | `/orders/:id`       | Pobierz zamówienie po ID      |
| POST   | `/orders/:id/ship`  | Zrealizuj (wyślij) zamówienie |

### `cmd/api/main_test.go`
Testy jednostkowe dla warstwy routingu. Sprawdzają:
- poprawną odpowiedź endpointu `/health`,
- czy wszystkie wymagane trasy zostały zarejestrowane w routerze.

Testy korzystają z `httptest` i nie wymagają uruchomionej bazy danych.

---

### `internal/config/config.go`
Prosta konfiguracja aplikacji. Struktura `Config` przechowuje:
- `AppPort` — port serwera HTTP (domyślnie `8081`),
- `DatabaseURL` — connection string do PostgreSQL.

> ⚠️ **Uwaga techniczna:** Wartości są na razie zahardkodowane w funkcji `Load()`. To celowy skrót — docelowo konfiguracja powinna być wczytywana ze zmiennych środowiskowych (np. przy pomocy biblioteki `godotenv` lub standardowego `os.Getenv`).

---

### `internal/db/db.go`
Moduł odpowiedzialny wyłącznie za nawiązanie połączenia z PostgreSQL. Używa sterownika `pgx` (via `database/sql`). Funkcja `Connect` zwraca `*sql.DB`, który jest następnie przekazywany do wszystkich modułów domenowych.

---

### `internal/product/product.go`
Moduł zarządzający produktami. Zawiera:
- **Model** `Product` z polami: `ID`, `ArticleCode`, `Name`, `CreatedAt`.
- **Strukturę `DB`** — opakowanie na `*sql.DB` będące zarówno repozytorium, jak i handlerem HTTP (więcej w sekcji o architekturze).
- **Metody bazodanowe:** `CreateProduct`, `ListProducts`.
- **Handlery HTTP:** `Create` (POST `/products`), `List` (GET `/products`).

Handler `Create` waliduje wejście: sprawdza poprawność JSON-a, usuwa białe znaki i wymaga niepustego `articleCode` oraz `name`.

### `internal/product/product_test.go`
Testy jednostkowe przy użyciu `go-sqlmock`. Pokrywają:
- odpowiedź `400 Bad Request` przy błędnym JSON-ie,
- poprawne tworzenie produktu (`201 Created`),
- poprawne listowanie produktów (`200 OK`).

---

### `internal/stock/stock.go`
Moduł zarządzający stanem magazynowym. Zawiera:
- **Modele:** `ReportItem`, `InboundRequest`, `InboundResponse`.
- **Metody bazodanowe:**
  - `ProductExists` — sprawdza istnienie produktu przed przyjęciem towaru,
  - `AddInbound` — atomowo (w transakcji) aktualizuje tabelę `stock` i zapisuje wpis w `inbound_operations`,
  - `Report` — zwraca zestawienie stanu magazynowego (JOIN produktów ze stockiem).
- **Handlery HTTP:** `Inbound` (POST `/inbounds`), `ReportHTTP` (GET `/stock`).

Operacja przyjęcia towaru jest zabezpieczona transakcją bazodanową (`BEGIN / COMMIT / ROLLBACK`), by uniknąć niespójności danych przy równoległych żądaniach.

### `internal/stock/stock_test.go`
Testy jednostkowe przy użyciu `go-sqlmock`. Pokrywają:
- odpowiedź `400 Bad Request` przy błędnym JSON-ie,
- kompletny scenariusz przyjęcia towaru (weryfikacja produktu + transakcja),
- endpoint raportu magazynowego.

---

### `internal/order/order.go`
Najbardziej złożony moduł — zarządza zamówieniami. Zawiera:
- **Stałe statusów:** `CREATED`, `SHIPPED`.
- **Błędy domenowe:** `ErrInsufficientStock`, `ErrAlreadyShipped`.
- **Modele:** `Order`, `OrderItem`, `CreateOrderRequest`, `ShipResponse`.
- **Metody bazodanowe:**
  - `GetStock` — pobiera aktualny stan dla produktu,
  - `CreateOrder` — tworzy zamówienie i jego pozycje w jednej transakcji,
  - `GetOrder` — pobiera zamówienie razem z pozycjami,
  - `ShipOrder` — realizuje zamówienie: blokuje wiersze (`FOR UPDATE`), weryfikuje stany magazynowe, odejmuje ilości ze stocku, aktualizuje status i zapisuje wpis w `outbound_operations`.
- **Handlery HTTP:** `Create`, `Get`, `Ship`.

Handler `Create` scala duplikaty pozycji (ten sam `productId` w kilku wierszach JSON-a jest sumowany) i wstępnie weryfikuje stany magazynowe przed zatwierdzeniem zamówienia.

### `internal/order/order_test.go`
Testy jednostkowe przy użyciu `go-sqlmock`. Pokrywają:
- odpowiedź `400 Bad Request` przy błędnym JSON-ie,
- pełny scenariusz tworzenia zamówienia,
- pobieranie zamówienia po ID,
- pełny scenariusz wysyłki zamówienia (transakcja z weryfikacją stocku),
- walidację helpera `parseID`.

---

### `migrations/001_init.sql`
Pierwsza migracja — tworzy pełny schemat bazy danych:
- `products` — rejestr produktów,
- `stock` — aktualny stan magazynowy (1:1 z produktem),
- `inbound_operations` — log operacji przyjęcia towaru,
- `orders` — zamówienia,
- `order_items` — pozycje zamówień,
- `outbound_operations` — log operacji wysyłki.

Migracja używa formatu **goose** (`-- +goose Up` / `-- +goose Down`).

### `migrations/002_add_order_description.sql`
Dodaje opcjonalną kolumnę `description TEXT` do tabeli `orders`. Jest to przykład ewolucji schematu bez niszczenia istniejących danych.

---

### `docker-compose.yml`
Uruchamia lokalną instancję PostgreSQL 16 w kontenerze Docker:
- użytkownik/hasło: `postgres/postgres`,
- baza: `mini_wms`,
- port: `5433` (celowo inny niż domyślny `5432`, by uniknąć konfliktu z lokalną instalacją).

---

### `go.mod` / `go.sum`
Pliki modułu Go. Kluczowe zależności:
- `github.com/gin-gonic/gin` — framework HTTP,
- `github.com/jackc/pgx/v5` — sterownik PostgreSQL,
- `github.com/DATA-DOG/go-sqlmock` — mockowanie bazy w testach.

---

## Decyzja architektoniczna — dlaczego jest tymczasowa?

### Stan obecny

W każdym module domenowym (`product`, `stock`, `order`) istnieje **jeden typ `DB`**, który łączy w sobie dwie odpowiedzialności:

1. **Dostęp do danych** — zapytania SQL, transakcje,
2. **Obsługa HTTP** — parsowanie żądań, walidacja, formowanie odpowiedzi JSON.

```
// Przykład z product.go — jedna struktura, dwie role:
type DB struct{ db *sql.DB }

func (d *DB) CreateProduct(...) (Product, error) { /* SQL */ }
func (d *DB) Create(c *gin.Context) { /* HTTP handler */ }
```

Takie podejście było **świadomym skrótem na potrzeby nauki** — pozwoliło skupić się na logice biznesowej i mechanizmach Go bez narzutu boilerplate'u.

### Dlaczego jest to problematyczne?

- **Naruszenie zasady pojedynczej odpowiedzialności (SRP)** — jeden typ robi zbyt wiele.
- **Trudniejsze testowanie** — handler HTTP jest ściśle sprzężony z `*sql.DB`, co wymusza użycie `go-sqlmock` nawet w testach warstwy HTTP.
- **Brak interfejsów** — nie ma możliwości prostej zamiany implementacji bazy danych bez modyfikacji handlerów.
- **Słaba czytelność** — granica między logiką domenową a transportem HTTP jest niewidoczna.

### Docelowa architektura

Prawidłowe warstwowanie powinno wyglądać następująco:

```
HTTP Handler  →  Service / Use-Case  →  Repository (interfejs)  →  DB (implementacja)
```

Każda warstwa byłaby osobnym typem, komunikującym się przez interfejsy.

---

## Pomysły na dalszy rozwój i refaktoryzację

### Refaktoryzacja architektury
- [ ] Wydzielić interfejsy repozytoriów (`ProductRepository`, `StockRepository`, `OrderRepository`) i oddzielić je od handlerów HTTP.
- [ ] Wprowadzić warstwę serwisową (`ProductService`, `OrderService`) skupiającą logikę biznesową.
- [ ] Przenieść handlery HTTP do osobnego pakietu (np. `internal/api` lub `internal/handler`).

### Konfiguracja
- [ ] Wczytywać konfigurację ze zmiennych środowiskowych (np. `os.Getenv` lub `github.com/joho/godotenv`).
- [ ] Dodać walidację konfiguracji przy starcie (np. sprawdzanie, czy `DATABASE_URL` jest ustawiony).

### Migracje
- [ ] Zintegrować runner migracji `goose` bezpośrednio z kodem startowym (`goose.Up`), by migracje były aplikowane automatycznie.

### Obsługa błędów
- [ ] Wprowadzić ujednolicone kody błędów domenowych i mapowanie ich na kody HTTP w jednym miejscu.
- [ ] Poprawić błąd w `ShipOrder` — błąd ze skanowania wiersza w pętli jest cicho pomijany.

### Testy
- [ ] Dodać testy integracyjne z prawdziwą bazą PostgreSQL (np. używając `testcontainers-go`).
- [ ] Zwiększyć pokrycie testami przypadków brzegowych (np. race condition przy wysyłce).

### Funkcjonalności
- [ ] Dodać paginację do endpointów listujących.
- [ ] Dodać endpoint do anulowania zamówień.
- [ ] Wprowadzić mechanizm rezerwacji stocku w momencie tworzenia zamówienia (zamiast dwuetapowej weryfikacji).
- [ ] Dodać middleware do logowania żądań i obsługi panics (`gin.Recovery`).
- [ ] Rozważyć uwierzytelnianie (np. API key lub JWT).

