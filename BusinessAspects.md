# Business Aspects

Journey-Service (https://developer.sbb.ch/apis/journey-service/documentation) is a routing service provided by SBB and is widely used by SBB public transportation (train, bus, tram,..) related applications, such as **[sbb.ch](https://www.sbb.ch/en/home.html)** or **SBB Mobile App**.

## Relevant Standards
Some important links:
* [BAV](https://www.bav.admin.ch/bav/de/home/verkehrsmittel/eisenbahn.html)
* [v580 FIScommun](https://www.allianceswisspass.ch/de/tarife-vorschriften/uebersicht/V580/Produkte-der-V580-FIScommun-1)
* [Open-Data Plattform Mobility Switzerland](https://opentransportdata.swiss/de/)
* [TransportMode](TransportMode.md)

## Design goals
### End-user consistency
According to **SBB KI strategy (de:Kundeninformation)** it is a **declared goal to communicate consistent public transport information** over all end-user channels!
**J-S plays an important role** to provide such consistency **to any passengers on any end-user API resp. UI-channel**.

**J-S consumers must be aware that they might violate these SBB business rules by ignoring, manipulating or redefining given values and may impact unwanted end-user or public media critics**.

Some expressions are even related to **conventions by [BAV](https://www.bav.admin.ch/bav/de/home/verkehrsmittel/eisenbahn.html), SBB Infrastructure, SBB Personenverkehr or public transport Switzerland** in general.

## About specific datatypes
### Place
In J-S `Place` is an abstract type and supertype of concrete `StopPlace`, `Address` and `PointOfInterest`.
All data are rather masterdata with rare refresh cycles (but they happen, especially for new yearly planning period for StopPlace).

Relevant systems dealing with StopPlace's:
1. The core system to manage all StopPlace data is **[DiDok](https://developer.sbb.ch/apis/servicepoints/information)** (validity, unique naming, other properties).
2. The system containing all planned journeys for a certain period for known `ServiceProduct's` is **INFO+**. StopPlace's from DiDok are related on planned journeys for a certain period (for e.g. 2022, from Bern to Zürich HB as IC 1 713, where 'Bern' and 'Zürich HB' are such StopPlace's).
3. The journey-planner system to route `Trip's` is Hafas, based on INFO+ (planned journeys) and **CUS** (realtime changes on planned journeys, see below [Realtime behaviour](#realtime-behaviour).

**Important to understand:**
* `/v3/places` and `/v3/trips` request on Hafas, therefore the contained Place's`are only those actively routed by Hafas:
    * Some StopPlace's have 2 "equivalent" UICs (for e.g. on geographical border-points like a Swiss UIC and a foreign country UIC), but Hafas knows only ONE of those two, though both ar given by DiDok
    * `PointOfInterest`places are originally provided by [Journey-POIs-Service](https://developer.sbb.ch/apis/journey-pois/information) and 1:1 resused by Hafas routing.
    * `Address` places are based on a postal directory refreshed in regular cycles.
    *  **_v3/places_** are considered **independent of their validFrom/To time-frame**, by means Hafas might find and route such places, even if they are meant for a future date even if the Trip dateTime is before.
    *  **_v3/stop-places_** is a Journey-Service internal subset of **actively routed** `StopPlace`s by Hafas, that **considers validFrom/To according to DiDok** master. Be aware not all DiDok StopPlace's are contained.

### VehicleMode (or TransportMode)

v3 `VehicleMode`

See [TransportMode](TransportMode.md)

There are specific extensions for developer convenience, such as:
* TransportProductV2::vehicleIconName showing the appropriate name in **SBB Corporate-Identity resources** (though the resource itself must be allocated by the consumer)
* the developer is still free whether he prefers its own mapping from TransportProductV2::vehicleType

## Business logic aspects
### Translations
Some properties' resp. their value-expressions might be **translated according to requested "Accept-language" to german (de), french (fr), italian (it) and english (en)** for e.g.:
* Station-Names in request accept all 4 languages usually, though the reply (StopV2::name) contains only the local Switzerland translation as a special case (Geneva → Genève)
* v3.Notice::value or v2.Note::value are sometimes translated by SBB P Data-Mgmt
* Translations with a standard (TransportProductV2::trackTranslation) and short-translation (TransportProductV2::trackTranslationShort):
* other texts are translated by SBB Business Rules, like StopV2::*DelayText

### Formatted fields
J-S sometimes provides fields with a "*Formatted" suffix which contain values, that must be showed to public end-users instead of alternatively declared field without the suffix for SBB internal usage only, for e.g.
* TransportProductV2::number → B2E only: meant for SBB internal services or employees (for e.g. to display in SBB Casa to be seen by SBB employees only)
* TransportProductV2::number**Formatted** → B2C or B2P: Business Rule impacted value for end-users (for e.g. to display in SBB Webshop, SBB Mobile, ..)

### Realtime behaviour
The SBB underlying systems may **provide realtime-data, typically TODAY only (~ NOW..+4h)**.

Getting the right realtime conclusions can be tricky, therefore J-S provides convenience data whenever possible.

`ScheduledStopStatus` for e.g. contains pre-calculated fields to inform about relevant realtime status of a TransportProductV2 at a specific StopV2:
* ::arrivalTimeAimed/Rt, departureTimeAimed/Rt
* ::stopStatus s. [Journey-Service_Routing-Basics](https://github.com/SchweizerischeBundesbahnen/journey-service/blob/master/Journey-Service_Routing-Basics.pdf)
* ::forBoarding, ::forAlighting

About any ***Rt** properties:
* Ideally these fields are always null, means transport organisations are operating as planned
* If any vehicle (TransportProductV2) is not operating according to its scheduled plan, *Rt fields may contain correcting values here and there (availability usually max 2h in the future and may disappear quickly in the past, because irrelevant for the current instant in time)
* *Rt fields may update their values for the same trip or journey if repeatedly requested, since they express “real-time” behaviour. (However do update your query as less as possible, for performance reasons.)
* If the *Rt fields are empty, just use the corresponding (same name) fields without “Rt” suffix for properly planned values

Journey-Service does not know the exact position of a vehicle yet and does not even guarantee that a vehicle has passed a station in reality. (However we have stories to transmit such additional info in the near future.)

Remark:
* SBB staff see [StopPlace/Station state]( based on /display/FAHRPLAN/Haltestellen-Status)

## Most typical "Use cases"
### Trip-Request
Most consumers will probably be interested in a simple train-connection from A to B. This scenario is supported in 2 steps usually:
1. find the relevant origin and destination Place
2. find the `Trip` connection between those

* see [v2 APIs](v2/V2_APIs.md)
