# README #

Diese Spezifikation beschreibt die REST-Schnittstellen des **Bike Maintainer** Backends – einer Anwendung
zur Digitalisierung von Wartungsplänen für Motorräder und Autos. Sie deckt die Verwaltung von Fahrzeugen,
deren Wartungsaufgaben (Intervalle nach km und/oder Monaten) sowie die Wartungshistorie ab.

Das Modul enthält ausschließlich die OpenAPI-Definition (`src/main/resources/maintenance-api.yaml`) und wird
per `maven-remote-resources-plugin` als Artefakt gebündelt, sodass sie von konsumierenden Projekten
(z. B. `maintainer-backend`) eingebunden werden kann. Dort wird auf Basis dieser Spezifikation mit dem
`openapi-generator-maven-plugin` (Generator `spring`, `interfaceOnly`) der Server-Code (API-Interfaces und
Modelle) erzeugt.

## Architektur-Prinzipien ##

### Naming Convention ###

+ Property Names: Durchgehend camelCase (z. B. currentMileage, mileageAtPerformed).
+ Schemas: Benannt nach dem Muster {Entity}Request / {Entity}Response (z. B. VehicleRequest, VehicleResponse).
+ Pfade: Ressourcen liegen unterhalb von `/api/v1/vehicles`; MaintenanceTasks und MaintenanceLogs sind als
  Sub-Ressourcen eines Vehicles modelliert (`/api/v1/vehicles/{vehicleId}/maintenance-tasks`, `.../maintenance-logs`).

### Datentypen & Formate ###

+ IDs: Alle IDs (vehicleId, taskId, logId, ...) sind `integer` mit `format: int64` (DB-Autoincrement), keine UUIDs.
+ Timestamps: Erzeugungs-/Änderungszeitpunkte (createdAt, updatedAt) nutzen `date-time` (ISO-8601). Reine
  Datumsangaben ohne Uhrzeit (z. B. performedAt) nutzen `date`.
+ Enums: Feste Wertebereiche werden als `enum` modelliert (z. B. VehicleType: MOTORCYCLE, CAR).
+ Distanzen: Kilometerstände und -intervalle (currentMileage, intervalKm, mileageAtPerformed, ...) sind
  `integer` und werden in Kilometern angegeben.

### Datenqualität (Validation) ###

+ Required-Felder: Nur fachlich zwingende Felder sind als `required` markiert (z. B. name/type bei Vehicle,
  performedAt/mileageAtPerformed bei MaintenanceLog); rein optionale Angaben (make, model, description, notes, ...)
  bleiben bewusst optional.
+ Constraints: `maxLength`-Vorgaben orientieren sich an den erwarteten Feldlängen im Backend (z. B. name: 255,
  description/notes: 1000).

## Ressourcen ##

+ **Vehicles** (`/api/v1/vehicles`) – CRUD für Fahrzeuge (Motorrad/Auto) inkl. aktuellem Kilometerstand.
+ **MaintenanceTasks** (`/api/v1/vehicles/{vehicleId}/maintenance-tasks`) – Wartungsaufgaben eines Fahrzeugs mit
  km- und/oder monatsbasiertem Intervall.
+ **MaintenanceLogs** (`/api/v1/vehicles/{vehicleId}/maintenance-logs`) – Historie durchgeführter Wartungen
  (Datum, Kilometerstand, erledigte Tasks).

## Build ##

```bash
mvn clean package
```

Erzeugt ein Jar, das die OpenAPI-Spezifikation als Ressource enthält und in anderen Modulen als Maven-Abhängigkeit
eingebunden werden kann.
