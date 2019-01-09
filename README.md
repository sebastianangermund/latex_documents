# Timeseries
Service responsible for storing and retrieving timeseries data.

Run with:

`$ nameko run timeseries.timeseries --config config.yml`

## API
- Pub/Sub
    - [Store indoor reading](#markdown-header-store-indoor-reading)
    - [Store uc reading](#markdown-header-store-uc-reading)
    - [Store heating control](#markdown-header-store-heating-control)
    - [Store control state](#markdown-header-store-control-state)
    - [Store estimated indoor temps](#markdown-header-store-estimated-indoor-temps)
    - [Store follow up entry](#markdown-header-store-follow-up-entry)
    - [Store available power accumulation](#markdown-header-store-available-power-accumulation)
- RPC
    - [get_indoor_readings(m_id, s_id, history)](#markdown-header-get-indoor-readings)
    - [get_sensors_for_mid(m_id)](#markdown-header-get-sensors-for-measure-point)
    - [get_mean_t_for_mid(m_id, history, resolution)](#markdown-header-get-mean-temp-for-measure-point)
    - [get_mean_t_for_building(m_id, building_id, history, resolution)](#markdown-header-get-mean-temp-for-building)
    - [get_min_t_for_mid(m_id, history)](#markdown-header-get-min-temp-for-measure-point)
    - [get_min_t_for_building(m_id, building_id, history)](#markdown-header-get-min-temp-for-building)
    - [get_max_t_for_mid(m_id, history)](#markdown-header-get-max-temp-for-measure-point)
    - [get_max_t_for_building(m_id, building_id, history)](#markdown-header-get-max-temp-for-building)

### Pub/Sub

#### Store indoor reading

```python
store_indoor_reading(self, payload)
```

Listens to 'new_reading' from 'indoor_sensor_pipeline'.

Payload should look like:

``` python
{
    "mid": <m_id, str>,
    "sid": <s_id, str>,
    "bid": <b_id, int>,
    "ts": <timestamp>,
    "t": <float>,
    "h": <float>,
    "vs": <float>,
}
```

Output:

``` python
[{
    "measurement": "indoor_sensors",
    "tags": {
        "mid": <m_id, str>,
        "sid": <s_id, str>,
        "bid": <b_id, int>,
    },
    "time": <timestamp>,
    "fields": {
        "t": <float>,
        "h": <float>,
        "vs": <float>,
    }
}]
```

#### Store uc reading

```python
store_uc_reading(self, payload)
```

Listens to 'new_reading' from 'uc_sensor_pipeline'.

Payload should look like:

``` python
{
    "m_id": <m_id, str>,
    "ts":  <timestamp>,
    "readings": [{"name": "a", "value": 1}, {"name": "b", "value": 2}],
}
```

Output:

``` python
[{
    "measurement": "uc_sensors",
    "tags": {
        "mid": <m_id, str>,
    },
    "time": <timestamp>,
    "fields": {
        "a": 1.0,
        "b": 2.0,
    },
}]
```

#### Store heating control

```python
store_heating_control(self, payload)
```

Listens to 'new_control' from 'smart_heating'.

Minimum payload looks like this:

``` python
{
    "m_id": <m_id, str>,
    "time": <timestamp>,
}
```

Maximum output:

``` python
[{
    "measurement": "smart_heating_control",
    "tags": {
        "mid": <m_id, str>,
    },
    "time": <timestamp>,
    "fields": {
        "t_active": <float>,
        "t_offset": <float>,
        "e": <float>,
        "set_value": <float>,
        "repr_set_value": <float>,
        "it": <float>,
        "itp_it": <float>,
        "t_bias": <float>,
        "dt": <float>,
        "alpha_constant": <float>,
        "t_offset_optimization": <float>,
        "period": <int>,
        "state": <int>,
        "steady_duration": <int>,
        "end": <timestamp>,
        "dry_run": <None>,
        "sum_e": <float>,
        "pre_alpha_offset": <float>,
    },
}]
```

#### Store control state

```python
store_control_state(self, payload)
```

Listens to 'new_outdoor_temp_control' from 'smart_heating' and
'new_passive_outdoor_temp_control' from 'population'.

Example payload:

``` python
{
    "m_id": <m_id, str>,
    "time":  <timestamp>,
    "control_type": "passive",
    "end": <timestamp>, 
    "t_active": <timestamp>,
}
```

Output:

``` python
[{
    'measurement': 'control_state_history',
    'tags': {
        'mid': <m_id, str>,
    },
    'time': <timestamp>,
    'fields': {
        'state': 2,
        'end_time': <int>,
        't_active': <float>,
    },
}]
```

#### Store estimated indoor temps

```python
store_estimated_indoor_temps(self, payload)
```

Listens to 'calculate_step_response' from 'population'.

Payload should look like:

``` python
{
    "m_id": <m_id, str>,
    "estimated_indoor_temps": [
                {"ts": <timestamp>, "t": <float>},
                {"ts": <timestamp>, "t": <float>},
    ],
}
```

Output:

``` python
[
    {
        "measurement": "step_response_estimated_temps",
        "tags": {
            "mid": <m_id, str>,
        },
        "time": <int>,
        "fields": {
            "t": <float>,
        },
    },
    {
        "measurement": "step_response_estimated_temps",
        "tags": {
            "mid": <m_id, str>,
        },
        "time": <int>,
        "fields": {
            "t": <float>,
        },
    },
]
```

#### Store follow up entry

```python
store_follow_up_entry(self, payload)
```

Listens to 'dsm_job_follow_up_report' from 'population'.

Payload should look like:

``` python
{
    "cluster_id": <int>,
    "job_id": <int>,
    "time": <timestamp>,
    "power_qos": <float>,
    "rmse": <float>,
    "indoor_qos": <float>,
    "realized_power": <float>,
    "nominal_power": <float>,
    "ordered_power": <float>,
    "available_power": <float>,
}
```

Output:

``` python
[{
    "measurement": "dsm_follow_up",
    "tags": {
        "clusterid": <int>,
        "jobid": <int>,
    },
    "time": <timestamp>,
    "fields": {
        "power_qos": <float>,
        "rmse": <float>,
        "indoor_qos": <float>,
        "realized_power": <float>,
        "nominal_power": <float>,
        "ordered_power": <float>,
        "available_power": <float>,
    }
}]
```

#### Store available power accumulation

```python
store_available_power_accumulation(self, payload)
```

Listens to 'dsm_accumulate_available_power' from 'population'.

Payload should look like:

``` python
{
    "cluster_id": <int>,
    "job_id": <int>,
    "available_power": {
        <timestamp>: <float>,
        <timestamp>: <float>,
    }
}
```

Output:

``` python
[
    {
        "measurement": "dsm_follow_up",
        "tags": {
            "clusterid": <int>,
            "jobid": <int>,
        },
        "time": <int>,
        "fields": {'available_power': <float>},
    },
    {
        "measurement": "dsm_follow_up",
        "tags": {
            "clusterid": <int>,
            "jobid": <int>,
        },
        "time": <int>,
        "fields": {'available_power': <float>},
    },
]
```

### RPC

#### Get indoor readings

``` python
get_indoor_readings(m_id, s_id=None, history=24)
```

Returns dict with indoor readings for a specific measurepoint (m_id).

Optionally filter for a specific sensor (s_id).
Optionally extend or suspend the history, in hours, to return Defaults to 1440
minutes.

_Example output:_
``` python
{
    'abc123': [
        {'h': 0.27, 't': 21.3, 'time': '2017-05-13T11:04:36.752724992Z'},
        {'h': 0.27, 't': 21.3, 'time': '2017-05-13T11:04:43.783646976Z'},
        {'h': 0.27, 't': 21.3, 'time': '2017-05-13T11:04:45.337296896Z'},
        {'h': 0.27, 't': 21.3, 'time': '2017-05-13T11:04:46.569788928Z'}],
    'abc456': [
        {'h': 0.27, 't': 21.3, 'time': '2017-05-13T11:42:44.914573824Z'}]
}
```

#### Get sensors for measure point

``` python
get_sensors_for_mid(m_id)
```

Returns a list with sensors for a specific measurepoint (m_id).

_Example output:_
``` python
[
    'abc123',
    'abc456'
]
```

#### Get mean temp for measure point

As a list (for graph purposes):

``` python
get_mean_t_for_mid(m_id, history=1440, resolution=5)
```

Returns dict with mean indoor temperatures for a specific measurepoint (m_id)
with 5 min resolution.

Optionally extend or suspend the history, in minutes, to return Defaults to 1440
minutes.

Optionally set the resolution of the timeseries in minutes.
That is at what resolution to group the mean values. Defaults to 5 min.


_Example output:_
``` python
{
    '2017-05-18T01:30:00Z': 20.419422682539686,
    '2017-05-18T01:35:00Z': 20.319422682539686,
    ...
}
```

#### Get mean temp for building

As a list (for graph purposes):

``` python
get_mean_t_for_building(m_id, building_id, history=1440, group_by=5)
```

Returns dict with mean indoor temperatures for a specific measurepoint (m_id)
building (building_id) with 5 min resolution.

Optionally extend or suspend the history, in minutes, to return Defaults to 1440
minutes.

Optionally set the resolution of the timeseries in minutes.
That is at what resolution to group the mean values. Defaults to 5 min.


_Example output:_
``` python
{
    '2017-05-18T01:30:00Z': 20.419422682539686,
    '2017-05-18T01:35:00Z': 20.319422682539686,
    ...
}
```

#### Get min temp for measure point

``` python
get_min_t_for_mid(m_id, history=1440)
```

Returns dict with min indoor temperature and timestamp for a specific measurepoint (m_id).

Optionally extend or suspend the history, in hours, to return Defaults to 1440
minutes.

_Example output:_
``` python
{'time': '2017-05-18T07:50:31.059305984Z', 'min': 16.330353}
```

#### Get min temp for building

``` python
get_min_t_for_building(m_id, building_id, history=1440)
```

Returns dict with min indoor temperature and timestamp for a specific measurepoint (m_id) building (building_id).

Optionally extend or suspend the history, in hours, to return Defaults to 1440
minutes.

_Example output:_
``` python
{'time': '2017-05-18T07:50:31.059305984Z', 'min': 16.330353}
```

#### Get max temp for measure point

``` python
get_max_t_for_mid(m_id, history=1440)
```

Returns dict with max indoor temperature and timestamp for a specific measurepoint (m_id).

Optionally extend or suspend the history, in hours, to return Defaults to 1440
minutes.

_Example output:_
``` python
{'time': '2017-05-18T07:50:31.059305984Z', 'max': 22.330353}
```

#### Get max temp for building

``` python
get_max_t_for_building(m_id, building_id, history=1440)
```

Returns dict with max indoor temperature and timestamp for a specific measurepoint (m_id) building (building_id).

Optionally extend or suspend the history, in hours, to return Defaults to 1440
minutes.

_Example output:_
``` python
{'time': '2017-05-18T07:50:31.059305984Z', 'max': 22.330353}
```

#### Get latest reading from sensor

``` python
get_latest_reading_from_sensor(sensor_id)
```

Returns a dict with the latest reading from a sensor.

``` json
{
  "time": 1494673476752724992, // timestamp in UTC
  "t": 21.3, // temperature
  "h": 0.28 // humidity
}
```

## Influx Conf

Setting up a new database from scratch requires you to create the database and
user manually. The service is taking care of the rest. Follow the below steps to
create a database and users. Please change the passwords though :)

``` SQL
CREATE DATABASE timeseries
CREATE USER "admin" WITH PASSWORD '<password>' WITH ALL PRIVILEGES
USE timeseries
CREATE USER influx WITH PASSWORD '<password>'
GRANT READ ON timeseries TO influx
GRANT WRITE ON timeseries TO influx
PRECISION s
CREATE RETENTION POLICY a_year ON timeseries DURATION 8736h0m0s REPLICATION 1 DEFAULT
```
