# Docker Engine i PostgreSQL na WSL

## 1. Architektura

```text
Windows
└── Ubuntu WSL2
    └── Docker Engine
        └── kontener PostgreSQL
            └── baza wms_golang_db
```

Docker Engine, kontenery, obrazy i wolumeny znajdują się wewnątrz dystrybucji Ubuntu WSL. Nie są to zasoby wcześniejszej maszyny `podman-wsl`.

## 2. Instalacja Docker Engine w Ubuntu WSL

Uruchom właściwą dystrybucję z PowerShella, przykładowo:

```powershell
wsl -d Ubuntu-24.04
```

W Ubuntu wykonaj:

```bash
sudo apt update
sudo apt install -y docker.io docker-compose-v2
```

Uruchom usługę Docker:

```bash
sudo service docker start
```

Sprawdź instalację:

```bash
sudo docker version
sudo docker compose version
sudo docker run hello-world
```

Jeżeli Docker ma być używany bez `sudo`:

```bash
sudo usermod -aG docker "$USER"
newgrp docker
```

Grupa `docker` daje uprawnienia zbliżone do administratora systemu Linux.

## 3. Plik Compose dla PostgreSQL

W katalogu projektu utwórz lub uzupełnij `docker-compose.yml`:

```yaml
services:
  postgres:
    image: postgres:17
    container_name: postgres-db
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

Znaczenie nazw:

- `postgres-db`: nazwa kontenera.
- `wms_golang_db`: nazwa bazy danych wewnątrz PostgreSQL.
- `postgres-data`: trwały wolumen zawierający pliki bazy.
- `5432:5432`: port Ubuntu WSL i port PostgreSQL w kontenerze.

`external: true` wymaga wcześniejszego utworzenia wolumenu.

## 4. Utworzenie wolumenu

W Ubuntu WSL:

```bash
sudo docker volume create postgres-data
```

Sprawdzenie:

```bash
sudo docker volume ls
sudo docker volume inspect postgres-data
```

Docker przechowuje wolumeny wewnątrz Ubuntu, standardowo pod:

```text
/var/lib/docker/volumes/
```

Nie należy ręcznie edytować plików PostgreSQL w tym katalogu.

## 5. Uruchomienie PostgreSQL

Przejdź do projektu na dysku Windows:

```bash
cd /mnt/d/golang_upskilling/wms_golang
```

Uruchom Compose:

```bash
sudo docker compose up -d
```

Sprawdź kontener:

```bash
sudo docker compose ps
```

Sprawdź logi:

```bash
sudo docker compose logs postgres
```

Sprawdź gotowość bazy:

```bash
sudo docker exec postgres-db pg_isready -U admin -d wms_golang_db
```

Prawidłowy wynik:

```text
/var/run/postgresql:5432 - accepting connections
```

## 6. Sterowanie bazą

Uruchomienie usług Compose:

```bash
sudo docker compose up -d
```

Zatrzymanie bez usuwania kontenera:

```bash
sudo docker compose stop
```

Ponowne uruchomienie zatrzymanego kontenera:

```bash
sudo docker compose start
```

Zatrzymanie i usunięcie kontenera oraz sieci Compose:

```bash
sudo docker compose down
```

`docker compose down` nie usuwa zewnętrznego wolumenu `postgres-data`.

Nie wykonuj poniższej komendy, jeśli chcesz zachować dane:

```bash
sudo docker volume rm postgres-data
```

## 7. Connection string aplikacji Go

Jeżeli przekierowanie `localhost` z Windows do WSL działa:

```env
DATABASE_URL=postgres://admin:123@localhost:5432/wms_golang_db?sslmode=disable
```

Na komputerze firmowym `localhost:5432` może nie działać z powodu konfiguracji WSL, firewalla, VPN lub zasad bezpieczeństwa. Wtedy trzeba użyć adresu IP Ubuntu WSL.

Przykład:

```env
DATABASE_URL=postgres://admin:123@172.30.68.64:5432/wms_golang_db?sslmode=disable
```

`172.30.68.64` jest tylko przykładem. Należy zawsze sprawdzić aktualny adres.

## 8. Sprawdzenie aktualnego adresu IP Ubuntu WSL

W PowerShellu Windows:

```powershell
wsl -d Ubuntu-24.04 -- hostname -I
```

Jeśli `hostname` nie jest dostępne:

```powershell
wsl -d Ubuntu-24.04 -- sh -c "ip -4 addr show eth0"
```

Przykładowy wynik:

```text
inet 172.30.68.64/20 brd 172.30.79.255 scope global eth0
```

Właściwy adres to wartość po `inet`, bez `/20`:

```text
172.30.68.64
```

Nie używaj adresu po `brd`. Jest to adres rozgłoszeniowy sieci.

Sprawdź dostępność PostgreSQL z Windows:

```powershell
Test-NetConnection 172.30.68.64 -Port 5432
```

Oczekiwany wynik:

```text
TcpTestSucceeded : True
```

## 9. Ważne: ponowne ustawienie adresu po restarcie

Adres IP Ubuntu WSL może zmienić się po:

- restarcie Windows,
- wykonaniu `wsl --shutdown`,
- ponownym uruchomieniu dystrybucji Ubuntu,
- zmianie konfiguracji sieci, VPN lub WSL.

Jeżeli aplikacja Go ponownie zgłosi błąd połączenia, wykonaj poniższą procedurę.

### Krok 1: uruchom Docker i PostgreSQL

W Ubuntu WSL:

```bash
sudo service docker start
cd /mnt/d/golang_upskilling/wms_golang
sudo docker compose up -d
sudo docker compose ps
```

### Krok 2: sprawdź, czy PostgreSQL działa

```bash
sudo docker exec postgres-db pg_isready -U admin -d wms_golang_db
```

### Krok 3: pobierz nowy adres IP

W PowerShellu Windows:

```powershell
wsl -d Ubuntu-24.04 -- sh -c "ip -4 addr show eth0"
```

### Krok 4: sprawdź nowy adres

Podstaw znaleziony adres:

```powershell
Test-NetConnection NOWY_ADRES_IP -Port 5432
```

Przykład:

```powershell
Test-NetConnection 172.30.68.64 -Port 5432
```

### Krok 5: zaktualizuj `.env`

```env
DATABASE_URL=postgres://admin:123@NOWY_ADRES_IP:5432/wms_golang_db?sslmode=disable
```

Przykład:

```env
DATABASE_URL=postgres://admin:123@172.30.68.64:5432/wms_golang_db?sslmode=disable
```

### Krok 6: uruchom aplikację Go ponownie

Zamknij poprzedni proces i uruchom aplikację ponownie, aby ponownie wczytała `.env`.

```powershell
go run ./cmd/api
```

GoLand również wymaga zatrzymania i ponownego uruchomienia konfiguracji Run.

## 10. Szybki workflow developerski

### Start pracy

W Ubuntu WSL:

```bash
sudo service docker start
cd /mnt/d/golang_upskilling/wms_golang
sudo docker compose up -d
sudo docker compose ps
```

W PowerShellu Windows, jeżeli aplikacja nie łączy się z bazą:

```powershell
wsl -d Ubuntu-24.04 -- sh -c "ip -4 addr show eth0"
Test-NetConnection AKTUALNY_ADRES_IP -Port 5432
```

W razie zmiany adresu popraw `DATABASE_URL` w `.env` i ponownie uruchom aplikację.

### Koniec pracy

W Ubuntu WSL:

```bash
cd /mnt/d/golang_upskilling/wms_golang
sudo docker compose down
```

Opcjonalnie zatrzymaj całą dystrybucję z PowerShella:

```powershell
wsl --terminate Ubuntu-24.04
```

## 11. Podstawowa diagnostyka

Lista wszystkich kontenerów:

```bash
sudo docker ps -a
```

Lista wolumenów:

```bash
sudo docker volume ls
```

Port kontenera:

```bash
sudo docker port postgres-db
```

Logi PostgreSQL:

```bash
sudo docker logs postgres-db
```

Lista baz PostgreSQL:

```bash
sudo docker exec -it postgres-db psql -U admin -d postgres -c "\l"
```

Test aktualnej bazy:

```bash
sudo docker exec -it postgres-db psql -U admin -d wms_golang_db -c "SELECT current_database();"
```

## 12. Podman a Docker Engine

Docker Engine w Ubuntu WSL nie używa kontenerów ani wolumenów zapisanych wcześniej w `podman-wsl`.

```text
podman-wsl → osobne kontenery i wolumeny Podmana
Ubuntu WSL → osobne kontenery i wolumeny Dockera
```

Nawet wolumeny o identycznej nazwie `postgres-data` są oddzielnymi zasobami. Stare środowisko Podmana można usunąć dopiero po sprawdzeniu, że nowa baza Docker działa i nie zawiera potrzebnych danych wyłącznie w Podmanie.
