# Laboratorium 5

W ramach zadania utworzono dwie przestrzenie nazw:
- `ns-dev` – środowisko developerskie (mniejsze zasoby)
- `ns-prod` – środowisko produkcyjne (zasoby 2x większe niż w `ns-dev`)

W `ns-dev` skonfigurowano:
- limit maksymalnego zużycia zasobów dla pojedynczego kontenera: **0.2 CPU (200m)** oraz **256Mi RAM**
- możliwość uruchamiania aplikacji bez jawnych `requests/limits` (domyślne wartości z `LimitRange`)
- ograniczenie liczby Podów do **max 10**

Następnie uruchomiono trzy Deploymenty w `ns-dev`:
- `no-test` – celowo przekracza limity zasobów i **nie działa poprawnie**
- `yes-test` – spełnia wymagania i **działa poprawnie**
- `zero-test` – nie ma deklaracji `requests/limits`, **działa poprawnie**, a wartości są nadane automatycznie zgodnie z `LimitRange`

---

## Pliki w repo
- `00-namespaces.yaml` – tworzy przestrzenie nazw `ns-dev` i `ns-prod`
- `01-quota-ns-dev.yaml` – `ResourceQuota` dla `ns-dev` 
- `02-quota-ns-prod.yaml` – `ResourceQuota` dla `ns-prod` 
- `03-limitrange-ns-dev.yaml` – `LimitRange` dla `ns-dev` 
- `10-no-test.yaml` – Deployment `no-test` 
- `11-yes-test.yaml` – Deployment `yes-test` 
- `12-zero-test.yaml` – Deployment `zero-test` 

---

# 1) Utworzenie namespace oraz polityk zasobów

## 1.1) Utworzenie namespace
- `kubectl apply -f 00-namespaces.yaml`

## 1.2) Ustawienie Quota (ns-dev + ns-prod)
- `kubectl apply -f 01-quota-ns-dev.yaml`
- `kubectl apply -f 02-quota-ns-prod.yaml`

## 1.3) Ustawienie LimitRange w ns-dev
- `kubectl apply -f 03-limitrange-ns-dev.yaml`

---

# 2) Testy wymagane w zadaniu (ns-dev)

## 2.1) no-test (ma NIE działać)
- `kubectl apply -f 10-no-test.yaml`

## 2.2) yes-test (ma działać)
- `kubectl apply -f 11-yes-test.yaml`

## 2.3) zero-test (ma działać bez resources i dostać domyślne limity/requesty)
- `kubectl apply -f 12-zero-test.yaml`

---

# Dowody wykonania
## [SCREEN 1] Namespace’y zostały utworzone
Komenda:
- `kubectl get ns | egrep "ns-dev|ns-prod"`

Oczekiwane: `ns-dev` i `ns-prod` w stanie `Active`.

---

## [SCREEN 2] Quota w ns-dev 
Komenda:
- `kubectl -n ns-dev describe quota rq-ns-dev`

Oczekiwane: widoczne `pods: 10` oraz limity `requests/limits cpu/memory`.

---

## [SCREEN 3] Quota w ns-prod 
Komenda:
- `kubectl -n ns-prod describe quota rq-ns-prod`

Oczekiwane: limity 2x większe niż w `ns-dev`.

---

## [SCREEN 4] LimitRange w ns-dev
Komenda:
- `kubectl -n ns-dev describe limitrange lr-ns-dev`

Oczekiwane: widoczne `max cpu=200m memory=256Mi` oraz `defaultRequest` i `default`.

---

## [SCREEN 5] no-test nie działa poprawnie
Komendy:
- `kubectl -n ns-dev get pods -l app=no-test -o wide`
- `kubectl -n ns-dev get events --sort-by=.metadata.creationTimestamp | tail -n 30`
- `kubectl -n ns-dev describe deploy no-test`

Oczekiwane: błąd/odrzucenie przez LimitRange (np. przekroczone max CPU/RAM) lub brak uruchomionego Poda.

---

## [SCREEN 6] yes-test działa poprawnie (Running)
Komendy:
- `kubectl -n ns-dev get pods -l app=yes-test -o wide`
- `kubectl -n ns-dev describe pod -l app=yes-test`

Oczekiwane: Pod `Running` + requests/limits w zakresie 200m/256Mi.

---

## [SCREEN 7] zero-test działa i dostał domyślne request/limit z LimitRange
Komendy:
- `kubectl -n ns-dev get pods -l app=zero-test -o wide`
- `kubectl -n ns-dev describe pod -l app=zero-test`

Oczekiwane: Pod `Running`, a w `describe` widoczne automatycznie przypisane:
- Requests: `cpu 100m`, `memory 128Mi`
- Limits: `cpu 200m`, `memory 256Mi`

---

## [SCREEN 8] podsumowanie zasobów/zużycia Quota w ns-dev po uruchomieniach
Komenda:
- `kubectl -n ns-dev describe quota rq-ns-dev`

Oczekiwane: w sekcji `Used` widać zużycie zasobów przez działające Pody.

---

# [Potwierdzenie działania na zdjęciach np. 1.png-8.png w folderze "zrzuty ekranu"]

