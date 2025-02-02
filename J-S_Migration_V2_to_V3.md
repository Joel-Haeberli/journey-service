# Journey-Service (J-S) migration v2 to v3
Unfortunately this is a somewhat **time consuming migration**.
Related to Transmodel in v3, semantic changes and mappings come with some serious breaking API and Model changes in v3.

## SBB API Principles
see [SBB API Principles](https://schweizerischebundesbahnen.github.io/api-principles/) enforce [Zalando API Guidelines](https://opensource.zalando.com/restful-api-guidelines/), therefore some implementations aspects of v2 are refactored in v3:
* **Enum's** are consequently modeled as plain **x-tensible String's**
    * request enforces @Schema(allowableValues) closed list Enum
    * response 
    *    v2 declares Enum type
    *    v3 declares open list (extensible-enum-list) where null might be possible (in cases like irrelevant or not further distinguishable yet)
* **Boolean** won't be used as tri-state with null as valid option anymore [Zalando rule](https://opensource.zalando.com/restful-api-guidelines/#122)
* no more List<> are returned -> proper Response Object always

## Versioned URL
| Aspect | /v2 | /v3 |
|--------|-------|-----|
|API-Path|/b2c/v2/* | /v3/* |

## Which API-endpoints must be migrated
All greyed out APIs will be removed soon (~May 2023). Each of them appears deprecated and has a hint where to upgrade to:
![SWITCH hint](v2/resources/V2_toMigrate.png)

## Header

| Aspect | /v2 | /v3 |
|--------|-----------------------|-----|
|Request-Header	|like `Accept-Language` is used for standard or meta aspects (which are not in)|dito|
|Response-Header|like `Content-Language` (relates to `Accept-Language` or fallback-language by J-S)|dito|
|Tracing|`Request-Id` allows SBB internal Instana tracing|dito|

## Semantic type mappings

V2 was related to older Journey-Planner specifications and more proprietary than v3, which is based on the conceptual [Transmodel (CEN Europe) TRM6](https://www.transmodel-cen.eu/downloads/).

Remark:
* Obvious (quiet similar named) mappings are not mentioned explicitely below:

| J-S v3                   | J-S v2                          | J-A                           |
| ------------------------ | ------------------------------- |-----------------------------  |
| _Place_                  | see J-A                         | Location                      |
| ScheduledStopPoint       | StopV2                          | OrigDestType / StopType       |
| Point (GeoJSON)          | see J-A                         | CoordinatesWGS84              |
| LineString (GeoJSON)     | see J-A                         | Polyline                      |
| _ServiceJourneyPattern_  | see J-A                         | JourneySegment                |
| PTRideLeg                | LegV2 (::type=PUBLIC_TRANSPORT) | Leg (::type=PUBLIC_TRANSPORT) |
| AccessLeg                | LegV2 (::type=FOOTPATH)         | Leg (::type=FOOTPATH or CAR)  |
| PTConnectionLeg          | LegV2 (::type=TRANSFER)         | Leg (::type=TRANSFER)         |
| TripSummary              | TripSummaryV2                   | TripSummary                   |
| Notice                   | see J-A                         | Note                          |
| PTSituationMessage       | HimMessageV2                    | Message                       |
| DatedVehicleJourney      | see J-A                         | JourneyDetail                 |
| ServiceProduct           | TransportProductV2              | ProductType                   |
| OperatingPeriod          | ServiceDaysV2                   | ServiceDays                   |
| Problem + HttpStatus     | Problem + HttpStatus            | Throwable/Exception           |

## API's
| J-S v3                   | J-S v2                          | J-A                           |
| ------------------------ | ------------------------------- |-----------------------------  |
| /places                  | /locations                      | LocationAssistant             |
| /trips                   | /trips                          | TripAssistant                 |
| /vehicle-journeys        | /departures, /arrivals          | StationboardAssistant         |
|    "                     | /routes                         | RouteAssistant                |
| /situations              | /traffic                        | HimAssistant                  |
| /service-calendar        | /info                           | TimetableAssistant            |
| <N/A>                    | /trainFormation                 | TrainFormationFacade          |

## Generated ApiClient
OpenApi 3 based, see [Swagger2 -> OpenAPI 3](https://code.sbb.ch/projects/KI_FAHRPLAN/repos/journey-service/browse/journey-service-client/SwitchingSwagger2ToOpenApi3.md)
