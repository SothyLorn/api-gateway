# docker run
```bash
docker network create kong-net
```
```bash
docker run -d --name kong-database \
    --network=kong-net \
    -p 5432:5432 \
    -e "POSTGRES_USER=kong" \
    -e "POSTGRES_DB=kong" \
    -e "POSTGRES_PASSWORD=kong" \
    postgres:9.6
```
```bash
docker run --rm \
    --network=kong-net \
    -e "KONG_DATABASE=postgres" \
    -e "KONG_PG_HOST=kong-database" \
    -e "KONG_PG_USER=kong" \
    -e "KONG_PG_PASSWORD=kong" \
    -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
    kong:1.1.2 kong migrations bootstrap
```
```bash
docker run -d --name kong \
     --network=kong-net \
     -e "KONG_DATABASE=postgres" \
     -e "KONG_PG_USER=kong" \
     -e "KONG_PG_PASSWORD=kong" \
     -e "KONG_PG_HOST=kong-database" \
     -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
     -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
     -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
     -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
     -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
     -e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" \
     -p 80:8000 \
     -p 443:8443 \
     -p 8001:8001 \
     -p 8444:8444 \
     kong:1.1.2
```
```bash
docker run -p 1337:1337 -d --network="kong-net"  --name konga -e "NODE_ENV=production" -e "TOKEN_SECRET=abc123" pantsel/konga:0.14.1
```
# docker compose

```bash
version: "3.5"
services:
  kong-database:
    image: postgres:9.6.8-alpine
    hostname: kong-database
    container_name: kong-database
    restart: always
    environment:
      POSTGRES_USER: "kong"
      POSTGRES_DB: "kong"
    networks:
      - "kong-net"

  kong-migration:
    image: kong:1.1.2
    container_name: kong-migration
    restart: on-failure
    depends_on:
      - kong-database
    environment:
      KONG_DATABASE: "postgres"
      KONG_PG_HOST: "kong-database"
      KONG_CASSANDRA_CONTACT_POINTS: "kong-database"
      KONG_PG_USER: "kong"
      KONG_PG_PASSWORD: "kong"
    command: kong migrations bootstrap
    networks:
      - "kong-net"
  kong:
    image: kong:1.1.2
    hostname: kong
    container_name: kong
    restart: always
    ports:
      - "80:8000"
      - "443:8443"
      - "8001:8001"
      - "8444:8444"
    environment:
      KONG_DATABASE: "postgres"
      KONG_PG_HOST: "kong-database"
      KONG_CASSANDRA_CONTACT_POINTS: "kong-database"
      KONG_PROXY_ACCESS_LOG: "/dev/stdout"
      KONG_ADMIN_ACCESS_LOG: "/dev/stdout"
      KONG_PROXY_ERROR_LOG: "/dev/stderr"
      KONG_ADMIN_ERROR_LOG: "/dev/stderr"
      KONG_ADMIN_LISTEN: "0.0.0.0:8001, 0.0.0.0:8444 ssl"
    depends_on:
      - kong-migration
      - kong-database
    networks:
      - "kong-net"

  konga:
    image: pantsel/konga:0.14.1
    hostname: konga
    container_name: konga
    restart: always
    ports:
      - "1337:1337"
    environment:
      TOKEN_SECRET: "abc123"
      NODE_ENV: "production"
    networks:
      - "kong-net"

networks:
  kong-net:
    name: kong-net
    driver: bridge

```
# https redirect
```bash
local scheme = kong.request.get_scheme()
if scheme == "http" then
  local host = kong.request.get_host()
  local query = kong.request.get_path_with_query()
  local url = "https://" .. host ..query
  kong.response.set_header("Location",url)
  return kong.response.exit(302,url)
end
```
# www redirect
```bash
local host = kong.request.get_host() 
if host == "www.demo.sothy.tech" then 
  local host = "demo.sothy.tech" 
  local query = kong.request.get_path_with_query() 
  local url = "https://" .. host ..query 
  kong.response.set_header("Location",url) 
  return kong.response.exit(302,url) 
end
```
