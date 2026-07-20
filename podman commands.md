
| command                                                                                                                      | description                              |
| ---------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------- |
| `podman volume ls`                                                                                                           | lista wszystkich wolumenów               |
| `podman volume inspect postgres-data`                                                                                        | szczegóły konkretnego wolumentu          |
| `podman ps -a --filter volume=postgres-data`                                                                                 | które kontenery używają wolumenu         |
| `podman ps`                                                                                                                  | listuje działające kontenery             |
| `podman ps -a`                                                                                                               | listuje wszystkie kontenery              |
| `podman exec -it postgres-db psql -U admin -d postgres -c "\l"`                                                              | listuje wszystkie bazy danych w postgres |
| `podman exec -it postgres-db psql -U admin -d postgres -t -c "SELECT datname FROM pg_database WHERE datistemplate = false;"` | listuje tylko nazwy baz                  |
| `podman exec -it postgres-db psql -U admin -d wms_golang_db -c "\dt"`                                                        | listuje tabele w konkretnej bazie danych |
|                                                                                                                              |                                          |
|                                                                                                                              |                                          |
