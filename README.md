# Sunrise API

StepZen's custom directive `@sequence` allows for multiple queries to be executed in a sequence, one after the other to return a single result. Each step in the sequence passes data to the next, allowing data from step 1 in a sequence to be used as arguments to step 2 in a sequence and so on.

This can allow you to create complex queries without having to manually orchestrate APIs, write lots of server-side logic, or handle asynchronous calls and database queries. In this example we will see how to use `@sequence` to query three APIs to get the sunrise based on the users's location (latitude and longitude) which we get using their IP address.

## Our three APIs

We'll use three APIs to get the sunrise based on the location (latitude and longitude) which we get using the IP address:

* [IP Geolocation API](https://www.abstractapi.com/ip-geolocation-api)
* [TimeZoneDB API](https://timezonedb.com/api)
* [Sunrise Sunset API](https://sunrise-sunset.org/api)

This requires executing a sequence of steps:

1. Get the `location` given an IP address.
2. Get `sunrise` from the `latitude` and `longitude` derived from the `location`.

### `SunriseSunset` contains properties about the sunrise and sunset

```graphql
type SunriseSunset {
  sunrise: String!
  sunset: String!
  solar_noon: String!
  day_length: String!
  civil_twilight_begin: String!
  civil_twilight_end: String!
  nautical_twilight_begin: String!
  nautical_twilight_end: String!
  astronomical_twilight_begin: String!
  astronomical_twilight_end: String!
  sunrise_unix: Int
  sunset_unix: Int
  abbreviation: String
}
```

### `Geolocation` contains properties about the location (i.e. longitude and latitude)

```graphql
type Geolocation {
  ip_address: String!
  city: String
  city_geoname_id: Int
  region: String
  region_iso_code: String
  region_geoname_id: Int
  postal_code: String
  country: String
  country_code: String
  country_geoname_id: Int
  country_is_eu: Boolean
  continent: String
  continent_code: String
  continent_geoname_id: Int
  longitude: Float
  latitude: Float
  abbreviation: String
}
```

### `Timezone` contains properties about the timezone

```graphql
type Timezone {
  countryCode: String
  countryName: String
  zoneName: String!
  abbreviation: String
  gmtOffset: Int!
  dst: Int
  zoneStart: Int
  zoneEnd: Int
  nextAbbreviation: String
  timestamp: Int
  timestring: String
  formatted: Date
}
```

## Queries

### `getLocationByIpAddress` gets a location using IP Geolocation API based upon a provided IP address

```graphql
type Query {
  getLocationByIpAddress(
    ip_address: String!
  ): Geolocation
    @rest(
      endpoint: "https://ipgeolocation.abstractapi.com/v1/?api_key=$api_key&ip_address=$ip_address"
      configuration: "abstractapi_config"
    )
}
```

### `getTimezoneByLatLong` gets the timezone from TimeZoneDB based upon a provided latitude and longitude

```graphql
type Query {
  getTimezoneByLatLong(
    latitude: Float!,
    longitude: Float!
  ): Timezone
    @rest(
      endpoint: "http://api.timezonedb.com/v2.1/get-time-zone?format=json&key=$api_key&by=position&lat=$latitude&lng=$longitude"
      configuration: "timezone_config"
    )
}
```

### `getSunriseByLatLong` gets the sunrise from the Sunrise Sunset API based upon a provided latitude and longitude

```graphql
type Query {
  getSunriseByLatLong(latitude: Float!, longitude: Float!): SunriseSunsetFoo
    @rest(
      endpoint: "https://api.sunrise-sunset.org/json?lat=$latitude&lng=$longitude"
      resultroot: "results"
      cel: """
      function transformREST(json)
      { ... }
      """
    )
}
```

### `collectSunrise` uses the `echo` connector to return the final result

```graphql
type Query {
  collectSunrise(
    sunrise: String!
    sunset: String!
    solar_noon: String!
    day_length: String!
    civil_twilight_begin: String!
    civil_twilight_end: String!
    nautical_twilight_begin: String!
    nautical_twilight_end: String!
    astronomical_twilight_begin: String!
    astronomical_twilight_end: String!
    sunrise_unix: Int
    sunrise_local: String
    sunset_unix: Int
    sunset_local: String
    abbreviation: String
  ): SunriseSunset
    @connector(type: "echo")
}
```

## `getSunriseByIp` sequence

```graphql
getSunriseByIp(ip_address: String!): SunriseSunset
  @sequence(
    steps: [
      { query: "getLocationByIpAddress" }
      { query: "getTimezoneByLatLong" }
      { query: "getSunriseByLatLong" }
      { query: "collectSunrise" }
    ]
  )
```

### Deploy the schema with stepzen start

Create a `config.yaml` file with your API keys included where it says `xxxxx`.

```yaml
configurationset:
  - configuration:
      name: abstractapi_config
      api_key: xxxx
  - configuration:
      name: timezone_config
      api_key: xxxx
```

Deploy the schema using the `stepzen start` command and issue a query in the StepZen Schema explorer. You can find your own IP address by Googling, "get my IP address."

```graphql
query MyQuery {
  getSunriseByIp(ip_address: "") {
    abbreviation
    sunrise
    sunset
  }
}
```


```graphql
getSunriseByIp(ip_address: String!): SunriseSunset
  @sequence(
    steps: [
      { query: "getLocationByIpAddress" }
      { query: "getTimezoneByLatLong" }
      { query: "getSunriseByLatLong" }
      { query: "collectSunrise" }
    ]
  )
```

## Sign Up

You can sign up for StepZen at the [following link](https://stepzen.com/api/auth/login).
