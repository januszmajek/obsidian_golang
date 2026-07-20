# PostgreSQL na WSL z Podmanem

Tu są **3 różne poziomy nazw i uruchamiania**, które łatwo pomylić.

## 1. `postgres-db` a `wms_golang_db`

- **`postgres-db`**: nazwa kontenera Podmana.
- **`wms_golang_db`**: nazwa bazy danych wewnątrz PostgreSQL.
- **`postgres-data`**: nazwa wolumenu przechowującego dane.

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

Uruchamia środowisko Linux WSL2, w którym działają kontenery.
```powershell
podman machine start podman-wsl
```

Uruchamia serwer PostgreSQL z bazą `wms_golang_db`.
```powershell
podman start postgres-db
```

Sprawdzenie, czy baza działa (`postgres-db` powinien mieć status `Up`)
```powershell
podman ps
```
## Typowe zakończenie pracy

```powershell
podman stop postgres-db
podman machine stop podman-wsl
```

Zatrzymanie kontenera **nie usuwa danych**, ponieważ pozostają w wolumenie `postgres-data`.
