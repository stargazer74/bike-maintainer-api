# README #

This specification describes the REST interfaces of the **Bike Maintainer** backend – an application for
digitizing maintenance schedules for motorcycles and cars. It covers the management of vehicles, their
maintenance tasks (intervals based on km and/or months), and the maintenance history.

The module contains only the OpenAPI definition (`src/main/resources/maintenance-api.yaml`) and is bundled
as an artifact via the `maven-remote-resources-plugin`, so it can be included by consuming projects
(e.g. `maintainer-backend`). There, the server code (API interfaces and models) is generated from this
specification using the `openapi-generator-maven-plugin` (generator `spring`, `interfaceOnly`).

## Architecture Principles ##

### Naming Convention ###

+ Property names: Consistently camelCase (e.g. currentMileage, mileageAtPerformed).
+ Schemas: Named according to the pattern {Entity}Request / {Entity}Response (e.g. VehicleRequest, VehicleResponse).
+ Paths: Resources are located under `/api/v1/vehicles`; MaintenanceTasks and MaintenanceLogs are modeled as
  sub-resources of a Vehicle (`/api/v1/vehicles/{vehicleId}/maintenance-tasks`, `.../maintenance-logs`).

### Data Types & Formats ###

+ IDs: All IDs (vehicleId, taskId, logId, ...) are `integer` with `format: int64` (DB auto-increment), not UUIDs.
+ Timestamps: Creation/modification timestamps (createdAt, updatedAt) use `date-time` (ISO-8601). Plain dates
  without a time component (e.g. performedAt) use `date`.
+ Enums: Fixed value ranges are modeled as `enum` (e.g. VehicleType: MOTORCYCLE, CAR).
+ Distances: Mileage readings and intervals (currentMileage, intervalKm, mileageAtPerformed, ...) are
  `integer` and expressed in kilometers.

### Data Quality (Validation) ###

+ Required fields: Only fields that are functionally mandatory are marked as `required` (e.g. name/type for
  Vehicle, performedAt/mileageAtPerformed for MaintenanceLog); purely optional attributes (make, model,
  description, notes, ...) remain deliberately optional.
+ Constraints: `maxLength` specifications are based on the expected field lengths in the backend (e.g. name: 255,
  description/notes: 1000).

## Resources ##

+ **Vehicles** (`/api/v1/vehicles`) – CRUD for vehicles (motorcycle/car) including current mileage.
+ **MaintenanceTasks** (`/api/v1/vehicles/{vehicleId}/maintenance-tasks`) – Maintenance tasks for a vehicle with
  km- and/or month-based intervals.
+ **MaintenanceLogs** (`/api/v1/vehicles/{vehicleId}/maintenance-logs`) – History of maintenance performed
  (date, mileage, completed tasks).

## Build ##

```bash
mvn clean package
```

Produces a jar that contains the OpenAPI specification as a resource and can be included as a Maven dependency
in other modules.
