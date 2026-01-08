# Düsseldorf Geocoding API

## Endpoint

**POST** `https://meta-puls.com/api/v1/geocode/dusseldorf`

---

## Request

**Headers:**
```
Content-Type: application/json
```

**Body:**
```json
{
  "address": "نام خیابان یا آدرس"
}
```

---

## Example

**Request:**
```bash
curl -X POST https://meta-puls.com/api/v1/geocode/dusseldorf \
  -H "Content-Type: application/json" \
  -d '{"address": "Königsallee"}'
```

**Response:**
```json
{
  "success": true,
  "data": {
    "query": "Königsallee",
    "formatted_address": "Königsallee, 40 Düsseldorf, Deutschland",
    "lat": 51.2218775,
    "lng": 6.779264,
    "x": 48869.9265,
    "y": -42589.4025,
    "place_id": "ChIJJ4K2Bj3KuEcR7JZ_1-86iIE",
    "google_maps_url": "https://www.google.com/maps?q=51.221877,6.779264"
  }
}
```

---

## Response Fields

| Field | Description |
|-------|-------------|
| `query` | عبارت جستجو شده |
| `formatted_address` | آدرس کامل از Google |
| `lat` | عرض جغرافیایی |
| `lng` | طول جغرافیایی |
| `x` | مختصات X در Unreal Engine (سانتی‌متر) |
| `y` | مختصات Y در Unreal Engine (سانتی‌متر) |
| `place_id` | شناسه مکان در Google |
| `google_maps_url` | لینک مستقیم به Google Maps |
