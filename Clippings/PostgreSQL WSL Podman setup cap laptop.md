# PostgreSQL na WSL z Podmanem

Tu są **3 różne poziomy nazw i uruchamiania**, które łatwo pomylić.

## 1. `postgres-db` a `wms_golang_db`

- **`postgres-db`**: nazwa kontenera Podmana.
- **`wms_golang_db`**: nazwa bazy danych wewnątrz PostgreSQL.
- **`postgres-data`**: nazwa woluymenu przechowującego dane.

### Czym jest wolumen?

Wolumen to trwałe miejsce na dane zarządzane przez Podmana. PostgreSQL zapisuje tam tabele, rekordy i konfigurację bazy. Dzięki wolumenowi dane pozostają dostępne po zatrzymaniu, ponownym uruchomieniu, a nawet usunięciu kontenera.

Usunięcie kontenera `postgres-db` nie usuwa danych z wolumenu `postgres-data`. Dane zostaną usunięte dopiero po usunięciu samego wolumenu.

```text
Kontener: postgres-db
└── PostgreSQL
    └── Baza: wms_golang_db

Dane zapisane w: postgres-data
```

Jeden kontener PostgreSQL może zawierać wiele baz danych.

## 2. Automatyczne uruchamianie

Obecnie **nie**. Nie podałeś polityki automatycznego restartu.

Dodatkowo muszą uruchomić się:

1. maszyna WSL Podmana,
2. kontener `postgres-db`.

Na razie uruchamiaj ręcznie:

```powershell
podman machine start podman-wsl
podman start postgres-db
```

Możesz później skonfigurować automatyczny start. Na początek ręczne sterowanie będzie prostsze.

## 3. Podstawowe komendy

### Uruchom Podmana/WSL

```powershell
podman machine start podman-wsl
```

### Zatrzymaj Podmana/WSL

```powershell
podman machine stop podman-wsl
```

### Uruchom bazę/kontener

```powershell
podman start postgres-db
```

### Zatrzymaj bazę/kontener

```powershell
podman stop postgres-db
```

### Zrestartuj bazę/kontener

```powershell
podman restart postgres-db
```

### Sprawdź działające kontenery

```powershell
podman ps
```

### Sprawdź wszystkie kontenery

```powershell
podman ps -a
```

### Wyświetl logi bazy

```powershell
podman logs postgres-db
```

### Usuń kontener

```powershell
podman rm -f postgres-db
```

## Typowy start pracy

```powershell
podman machine start podman-wsl
podman start postgres-db
podman ps
```

## Typowe zakończenie pracy

```powershell
podman stop postgres-db
podman machine stop podman-wsl
```

Zatrzymanie kontenera **nie usuwa danych**, ponieważ pozostają w wolumenie `postgres-data`.

## 4. Docker Compose i wpływ na codzienny workflow

`docker-compose.yml` lub `compose.yaml` opisuje kontener, jego konfigurację, porty i wolumeny w pliku. Podman może korzystać z tego pliku przez polecenie `podman compose`.

Compose nie zastępuje Podmana, WSL, PostgreSQL ani wolumenu. Zastępuje głównie długie polecenie `podman run` krótszym `podman compose up -d`.

### Zalecana konfiguracja

```yaml
services:
  postgres:
    image: postgres:17
    container_name: postgres-db-compose
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: "123"
      POSTGRES_DB: wms_golang_db
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data
    restart: unless-stopped

volumes:
  postgres-data:
    external: true
```

`external: true` oznacza użycie istniejącego wolumenu `postgres-data`. Dzięki temu Compose nie tworzy nowego, pustego wolumenu, a PostgreSQL korzysta z dotychczasowych danych.

### Przejście z ręcznie utworzonego kontenera

Istniejący kontener `postgres-db` i kontener z Compose nie mogą działać równocześnie na porcie `5432` ani korzystać jednocześnie z tego samego wolumenu PostgreSQL.

Zatrzymaj stary kontener:

```powershell
podman stop postgres-db
```

Nie musisz go od razu usuwać. Następnie w katalogu zawierającym plik Compose uruchom:

```powershell
podman compose up -d
```

Po sprawdzeniu, że Compose działa poprawnie, stary kontener możesz usunąć:

```powershell
podman rm postgres-db
```

Usunięcie kontenera nie usuwa danych z wolumenu `postgres-data`.

### Workflow z Compose

#### Start pracy

```powershell
podman machine start podman-wsl
podman compose up -d
podman ps
```

- `podman machine start podman-wsl` uruchamia środowisko WSL Podmana.
- `podman compose up -d` tworzy lub uruchamia usługi opisane w pliku Compose.
- `podman ps` pokazuje działające kontenery.

#### Logi PostgreSQL

```powershell
podman compose logs postgres
```

Logi śledzone na żywo:

```powershell
podman compose logs -f postgres
```

#### Zatrzymanie kontenerów bez ich usuwania

```powershell
podman compose stop
```

Ponowne uruchomienie:

```powershell
podman compose start
```

#### Zatrzymanie i usunięcie kontenerów Compose

```powershell
podman compose down
```

Polecenie usuwa kontenery i sieć Compose, ale nie usuwa zewnętrznego wolumenu `postgres-data`.

#### Koniec pracy wraz z zatrzymaniem maszyny Podmana

```powershell
podman compose down
podman machine stop podman-wsl
```

### Różnica w obsłudze

Bez Compose:

```powershell
podman start postgres-db
podman stop postgres-db
```

Z Compose:

```powershell
podman compose up -d
podman compose down
```

Po przejściu na Compose najlepiej zarządzać kontenerem PostgreSQL przez `podman compose`, zamiast mieszać polecenia tworzące kontener ręcznie i przez Compose.

### Connection string aplikacji Go

Przy powyższej konfiguracji i aplikacji Go uruchamianej bezpośrednio na Windows:

```env
DATABASE_URL=postgres://admin:123@localhost:5432/wms_golang_db?sslmode=disable
```

Jeżeli aplikacja Go zostanie później uruchomiona jako usługa w tym samym pliku Compose, hostem PostgreSQL będzie nazwa usługi `postgres`, nie `localhost`:

```env
DATABASE_URL=postgres://admin:123@postgres:5432/wms_golang_db?sslmode=disable
```

Nie uruchamiaj równocześnie dwóch kontenerów PostgreSQL korzystających z tego samego wolumenu `postgres-data`.

## 5. Problem z dostępem do PostgreSQL przez `localhost` na Windows

Możliwa sytuacja:

- kontener PostgreSQL działa,
- PostgreSQL przyjmuje połączenia wewnątrz kontenera,
- port `5432` jest opublikowany przez Podmana,
- aplikacja Windows nie może połączyć się przez `localhost:5432`.

Przykładowy błąd aplikacji Go:

```text
dial tcp 127.0.0.1:5432: connectex: No connection could be made because the target machine actively refused it
```

### Sprawdzenie PostgreSQL wewnątrz kontenera

```powershell
podman exec postgres-db pg_isready -U admin -d wms_golang_db
```

Prawidłowy wynik:

```text
/var/run/postgresql:5432 - accepting connections
```

Taki wynik oznacza, że baza działa, a problem dotyczy dostępu z Windows do WSL.

### Sprawdzenie dostępu przez `localhost`

```powershell
Test-NetConnection localhost -Port 5432
```

Jeżeli wynik zawiera:

```text
TcpTestSucceeded : False
```

Windows nie może połączyć się z portem wystawionym przez Podmana.

### Pobranie adresu IP maszyny WSL Podmana

```powershell
wsl -d podman-wsl -- sh -c "ip -4 addr show eth0"
```

Należy użyć adresu znajdującego się po słowie `inet`, bez maski sieci. Przykład:

```text
inet 172.30.68.64/20
```

W tym przykładzie właściwy adres to:

```text
172.30.68.64
```

Adres `172.30.79.255` jest adresem rozgłoszeniowym `brd`, więc nie należy używać go jako adresu bazy.

### Test bezpośredniego połączenia z WSL

```powershell
Test-NetConnection 172.30.68.64 -Port 5432
```

Jeżeli wynik zawiera:

```text
TcpTestSucceeded : True
```

aplikacja może tymczasowo łączyć się bezpośrednio z adresem WSL.

W pliku `.env`:

```env
DATABASE_URL=postgres://admin:123@172.30.68.64:5432/wms_golang_db?sslmode=disable
```

Po zmianie `.env` należy ponownie uruchomić aplikację Go.

> Adres IP WSL może zmienić się po restarcie WSL lub komputera. Po restarcie należy sprawdzić go ponownie i ewentualnie zaktualizować `.env`.

### Opcjonalne przekierowanie na `localhost`

W PowerShellu uruchomionym jako administrator:

```powershell
netsh interface portproxy add v4tov4 listenaddress=127.0.0.1 listenport=5432 connectaddress=172.30.68.64 connectport=5432
```

Sprawdzenie reguły:

```powershell
netsh interface portproxy show all
```

Sprawdzenie usługi wymaganej przez `portproxy`:

```powershell
Get-Service iphlpsvc
```

Jeżeli usługa nie działa:

```powershell
Start-Service iphlpsvc
Set-Service iphlpsvc -StartupType Automatic
```

Ponowny test:

```powershell
Test-NetConnection localhost -Port 5432
```

Jeżeli `localhost` nadal nie działa, możliwą przyczyną jest polityka bezpieczeństwa, firewall lub VPN na komputerze firmowym. Wtedy można użyć bezpośredniego adresu WSL albo zgłosić problem do działu IT.

### Usunięcie reguły `portproxy`

```powershell
netsh interface portproxy delete v4tov4 listenaddress=127.0.0.1 listenport=5432
```

### Diagnostyka po restarcie

```powershell
podman machine start podman-wsl
podman compose up -d
podman exec postgres-db pg_isready -U admin -d wms_golang_db
wsl -d podman-wsl -- sh -c "ip -4 addr show eth0"
Test-NetConnection localhost -Port 5432
```
