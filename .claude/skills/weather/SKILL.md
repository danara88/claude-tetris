---
name: weather
description: >
  Fetches and displays current local weather information using wttr.in (no API key needed).
  Use when the user asks about the weather, current temperature, forecast, or climate conditions
  in their location or any city. Triggers on: "clima", "weather", "temperatura", "pronóstico",
  "qué clima hace", "how's the weather", or /weather.
---

# Weather Skill

Fetch weather using `wttr.in` — no API key, no setup.

## How to Use

When the user asks about weather, determine the location and run the appropriate command.

### Get user's local weather (IP-based auto-detection)

```bash
curl -s "wttr.in/?format=v2"
```

### Get weather for a specific city

```bash
# Replace CITY with the city name (spaces as + or %20)
curl -s "wttr.in/CITY?format=v2"
```

### Get compact one-line summary

```bash
curl -s "wttr.in/CITY?format=3"
# Output example: Ciudad de México: ⛅️  +24°C
```

### Get full JSON data (for structured output)

```bash
curl -s "wttr.in/CITY?format=j1" | python3 -c "
import json, sys
d = json.load(sys.stdin)
cur = d['current_condition'][0]
area = d['nearest_area'][0]
city = area['areaName'][0]['value']
country = area['country'][0]['value']
temp_c = cur['temp_C']
feels = cur['FeelsLikeC']
desc = cur['weatherDesc'][0]['value']
humidity = cur['humidity']
wind = cur['windspeedKmph']
print(f'Location : {city}, {country}')
print(f'Condition: {desc}')
print(f'Temp     : {temp_c}°C (feels like {feels}°C)')
print(f'Humidity : {humidity}%')
print(f'Wind     : {wind} km/h')
"
```

## Workflow

1. If user provided a city/location → use that city in the URL.
2. If no location given → use `wttr.in/Saltillo,Coahuila,Mexico?format=j1` (default city).
3. Run the JSON command for structured output, then present results clearly.
4. If the user wants a forecast, add `&lang=es` for Spanish or parse `weather[0..2]` array from JSON for next 3 days.

### 3-day forecast

```bash
curl -s "wttr.in/CITY?format=j1" | python3 -c "
import json, sys
d = json.load(sys.stdin)
for i, day in enumerate(d['weather']):
    label = ['Hoy', 'Mañana', 'Pasado mañana'][i]
    date = day['date']
    max_c = day['maxtempC']
    min_c = day['mintempC']
    desc = day['hourly'][4]['weatherDesc'][0]['value']
    print(f'{label} ({date}): {desc}, {min_c}°C – {max_c}°C')
"
```

## Response Format

Present weather in a clean, human-readable format. Example:

```
📍 Ciudad de México, Mexico
🌤  Partly Cloudy
🌡  22°C (feels like 21°C)
💧 Humidity: 65%
💨 Wind: 18 km/h
```

Always answer in the same language the user used to ask.
