# Umweltnavi API Documentation

> [!WARNING]
> **Data rights have not been confirmed.** The data served by this API belongs to various German federal and state agencies. Licensing terms, usage restrictions, and redistribution rights have not been verified. Use at your own risk and do not assume the data is freely available for commercial or public use. Always check the `license` and `owner` fields on individual objects for attribution requirements.

The Umweltnavi API is the data backbone behind Germany's interactive environmental portal ([umweltportal.niedersachsen.de](https://umweltportal.niedersachsen.de)). It serves environmental, nature, and geographic data for multiple German federal states — from dragonfly sightings and nature reserves to wind turbines and flood zones.

## OpenAPI Specification

**[`umweltnavi-topics-api.yaml`](./umweltnavi-topics-api.yaml)** — the machine-readable OpenAPI 3.0.3 spec is the core of this repository. Use it to:

- **Generate API clients** in any language (Python, TypeScript, Java, ...) with tools like [openapi-generator](https://openapi-generator.tech/)
- **Import into Swagger UI / Redoc** for interactive, browsable API docs
- **Validate requests & responses** with schema-aware HTTP clients
- **Run automated tests** with [Schemathesis](https://github.com/schemathesis/schemathesis) (this spec was validated across 9 fuzzing iterations)
- **Load into Postman / Insomnia** for manual exploration

| | |
|---|---|
| **Base URL** | `https://api.umweltnavi.info/web/v1` |
| **Spec format** | OpenAPI 3.0.3 (YAML) |
| **Endpoints** | 6 |
| **States** | `ni` (Niedersachsen), `sh` (Schleswig-Holstein), `rp` (Rheinland-Pfalz) |
| **Schemas** | 25+ reusable component definitions |

---

## Table of Contents

1. [Quick Start](#1-quick-start)
2. [Authentication & Headers](#2-authentication--headers)
3. [Supported States](#3-supported-states)
4. [API Endpoints Overview](#4-api-endpoints-overview)
5. [Topics — The Content Hierarchy](#5-topics--the-content-hierarchy)
6. [Spatial Objects — Map Queries](#6-spatial-objects--map-queries)
7. [Object Detail — Deep Dive on a Feature](#7-object-detail--deep-dive-on-a-feature)
8. [Search — Full-Text Lookup](#8-search--full-text-lookup)
9. [Pages — Editorial Content](#9-pages--editorial-content)
10. [Events — Press Releases & News](#10-events--press-releases--news)
11. [Data Sources & Object ID Prefixes](#11-data-sources--object-id-prefixes)
12. [Error Handling](#12-error-handling)
13. [Known Limitations & API Quirks](#13-known-limitations--api-quirks)

---

## 1. Quick Start

Fetch all environmental topics for Niedersachsen:

```bash
curl 'https://api.umweltnavi.info/web/v1/ni/topics' \
  -H 'Origin: https://umweltportal.niedersachsen.de' \
  -H 'Accept: application/json'
```

Search for "Eisvogel" (kingfisher) sightings:

```bash
curl 'https://api.umweltnavi.info/web/v1/ni/search?q=eisvogel' \
  -H 'Origin: https://umweltportal.niedersachsen.de' \
  -H 'Accept: application/json'
```

Query spatial objects on a map tile near Schwerin:

```bash
curl 'https://api.umweltnavi.info/web/v1/ni/objects/16/53.7938/12.2032/53.7841/12.2444?tag=map-service%20OR%20spatial-object' \
  -H 'Origin: https://umweltportal.niedersachsen.de' \
  -H 'Accept: application/json'
```

---

## 2. Authentication & Headers

The API does **not** require authentication tokens or API keys. However, two headers are expected:

| Header   | Value | Purpose |
|----------|-------|---------|
| `Origin` | `https://umweltportal.niedersachsen.de` | CORS origin — required by the server |
| `Accept` | `application/json` | Ensures JSON responses (some errors default to HTML) |

---

## 3. Supported States

Every endpoint is scoped to a German federal state (Bundesland) via the `{state}` path parameter. Currently three states are supported:

| Code | State | Notes |
|------|-------|-------|
| `ni` | Niedersachsen | Most complete dataset — the primary portal |
| `sh` | Schleswig-Holstein | Full support, slightly different topic tree |
| `rp` | Rheinland-Pfalz | Full support |

> **Tip:** The topic tree, available categories, and number of objects differ between states. Niedersachsen (`ni`) has the richest data.

---

## 4. API Endpoints Overview

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | [`/{state}/topics`](#5-topics--the-content-hierarchy) | Full hierarchical topic tree |
| `GET` | [`/{state}/objects/{zoom}/{lat1}/{lon1}/{lat2}/{lon2}`](#6-spatial-objects--map-queries) | Spatial objects in a bounding box |
| `GET` | [`/{state}/objects/{objectId}`](#7-object-detail--deep-dive-on-a-feature) | Detailed info for a single object |
| `GET` | [`/{state}/search?q=...`](#8-search--full-text-lookup) | Full-text search across all objects |
| `GET` | [`/{state}/pages/{pageId}`](#9-pages--editorial-content) | CMS editorial content page |
| `GET` | [`/{state}/events`](#10-events--press-releases--news) | Press releases and news events |

---

## 5. Topics — The Content Hierarchy

```
GET /{state}/topics
```

Returns the complete topic tree that drives the portal's navigation and map layer catalog. This is the **most important endpoint** to understand — it defines what data exists in the portal.

### The 4-Level Hierarchy

```
Topic                          e.g. "Natur und Landschaft"
 └─ Subtopic                   e.g. "Schutzgebiete"
     └─ SelectcategoryGroup    e.g. "Natura 2000"
         └─ Selectcategory     e.g. "fauna-flora-habitat-gebiete"
             └─ Category       e.g. "naturschutzgebiet"
```

| Level | What it is | Example |
|-------|-----------|---------|
| **Topic** | Top-level theme, shown as main navigation tabs | "Natur und Landschaft", "Klimawandel" |
| **Subtopic** | Thematic section within a topic | "Schutzgebiete", "Oberflächengewässer" |
| **SelectcategoryGroup** | Logical grouping of map layers (display only) | "Natura 2000", "Erneuerbare Energien" |
| **Selectcategory** | A toggleable map layer in the portal | `libellen`, `windenergieanlage`, `naturschutzgebiete` |
| **Category** | Individual feature type within a layer | `gebaenderte-prachtlibelle`, `seehundliegeplatz` |

### Current Topic Tree (Niedersachsen)

The `ni` state has **7 top-level topics**:

| Topic ID | Name | Example Content |
|----------|------|-----------------|
| `natur-und-landschaft` | Natur und Landschaft | Nature reserves, biotopes, water bodies, geology |
| `freizeit-und-tourismus` | Freizeit und Tourismus | Swimming spots, hiking, bike routes, zoos |
| `gesundheit-risiken-und-sicherheit` | Gesundheit, Risiken und Sicherheit | Flood zones, nuclear safety, air quality, noise maps |
| `klimawandel` | Gesellschaft und Klimawandel | Energy, population stats, planning, municipalities |
| `pflanzen-und-tierwelt` | Pflanzen- und Tierwelt | Species of the year, wildlife sightings, wolves, lynx |
| `landwirtschaft-und-boden` | Landwirtschaft und Boden | Soil erosion, agricultural subsidies, livestock damage |
| `metadaten` | Metadaten | INSPIRE/OpenData metadata catalog |

### Key Fields on Selectcategories

Selectcategories carry configuration flags that control portal behavior:

| Field | Type | Meaning |
|-------|------|---------|
| `icon` | URL | Map marker icon for this layer |
| `color` | Hex color | `#4CAF50` — accent color for the layer |
| `photo_upload` | boolean | Users can upload photos for this category |
| `route_guidance` | boolean | Navigation/routing is available |
| `include_tip_of_day` | boolean | Shown in the "Tip of the Day" feature |
| `hidden` | boolean | Layer exists but is not shown in navigation |
| `pages` | integer[] | IDs of related editorial content pages |

### Example Response (abbreviated)

```json
[
  {
    "id": "natur-und-landschaft",
    "name": "Natur und Landschaft",
    "image": {
      "url": "https://api.umweltnavi.info/v2/i/340b3b13-....jpg",
      "title": "Themen Visual"
    },
    "subtopics": [
      {
        "id": "lebensraeume",
        "name": "Lebensräume",
        "info_text": "Entdecke natürliche und naturnahe Lebensräume...",
        "selectcategory_groups": [
          {
            "name": "Lebensräume für Tiere und Vögel",
            "selectcategories": [
              {
                "id": "seehundliegeplaetze",
                "name": "Seehundliegeplätze",
                "icon": "https://api.umweltnavi.info/v2/i/....svg",
                "color": "#03A9F4",
                "categories": [
                  { "id": "seehundliegeplatz", "name": "Seehundliegeplatz" }
                ]
              }
            ]
          }
        ]
      }
    ]
  }
]
```

---

## 6. Spatial Objects — Map Queries

```
GET /{state}/objects/{zoom}/{lat1}/{lon1}/{lat2}/{lon2}
```

The main geodata endpoint. Returns all spatial objects within a geographic bounding box, grouped into map tiles.

### Path Parameters

| Parameter | Type | Range | Description |
|-----------|------|-------|-------------|
| `state` | string | `ni`, `sh`, `rp` | Federal state |
| `zoom` | integer | 6–18 | Slippy map zoom level. Higher = more detail, smaller tiles |
| `lat1` | number | -85 to 85 | North edge latitude (WGS84) |
| `lon1` | number | -180 to 180 (exclusive) | West edge longitude (WGS84) |
| `lat2` | number | -85 to 85 | South edge latitude (WGS84) |
| `lon2` | number | -180 to 180 (exclusive) | East edge longitude (WGS84) |

> **Practical coordinate ranges for Germany:** Latitude 47.0–55.5, Longitude 5.5–15.5.
> Values outside this range are accepted but return empty tiles.

### Query Parameters (Filters)

| Parameter | Type | Example | Description |
|-----------|------|---------|-------------|
| `tag` | string | `spatial-object`, `map-service OR spatial-object` | Filter by tag. Supports `OR` operator |
| `selectcategory` | string | `libellen` | Show only objects of this selectcategory |
| `category` | string | `seehundliegeplatz` | Show only objects of this specific category |

### Response Structure

Results are organized into a **tile grid**. Each tile has X/Y coordinates in the [slippy map](https://wiki.openstreetmap.org/wiki/Slippy_map_tilenames) scheme and an array of objects:

```json
{
  "tiles": [
    {
      "x": 34989,
      "y": 21106,
      "objects": [
        {
          "id": "oorg-libellen-363663219",
          "type": "object",
          "name": "Gebänderte Prachtlibelle",
          "icon": "https://api.umweltnavi.info/v2/i/....svg",
          "category": "gebaenderte-prachtlibelle",
          "selectcategory": "libellen",
          "tags": ["spatial-object"],
          "shape": {
            "centroid": { "lat": 53.793045, "long": 12.203745 }
          }
        }
      ]
    },
    {
      "x": 34990,
      "y": 21106,
      "objects": []
    }
  ]
}
```

### How Tiles Work

The API uses the same tile coordinate system as OpenStreetMap. At zoom level 16, each tile covers roughly 600m x 600m. The response always includes tiles that intersect your bounding box — including empty ones.

| Zoom | Tile size (approx.) | Best for |
|------|-------------------|----------|
| 6 | ~600 km | Country overview |
| 10 | ~40 km | Regional view |
| 14 | ~2.5 km | City level |
| 16 | ~600 m | Neighborhood level |
| 18 | ~150 m | Street level |

### Example: Get all dragonfly sightings in an area

```bash
curl 'https://api.umweltnavi.info/web/v1/ni/objects/16/53.80/12.20/53.78/12.25?selectcategory=libellen' \
  -H 'Origin: https://umweltportal.niedersachsen.de' \
  -H 'Accept: application/json'
```

---

## 7. Object Detail — Deep Dive on a Feature

```
GET /{state}/objects/{objectId}
```

Returns the full detail for a single object. This is what the portal shows when you click on a map marker.

### What You Get

| Field | Description |
|-------|-------------|
| `name` | Human-readable name |
| `info` | Short info line (e.g. "gesichtet am 18.07.2025") |
| `description.text` | Full-text description (may be several paragraphs) |
| `images[]` | Array of photos with attribution |
| `data[]` | Grouped external links (Wikipedia, observation.org, etc.) |
| `pages[]` | IDs of related editorial pages (use `/pages/{id}` to fetch) |
| `shape` | Geographic shape — centroid, bounds, geometry type, polygon outline |
| `wms` | WMS map service reference for rendering the area on a map |
| `meta_data_url` | Link to external metadata record |
| `owner` / `license` | Data source attribution |

### Point vs. Polygon Objects

Objects come in two geometry types:

**Point objects** (species sightings, rescue points, energy installations):
```json
"shape": {
  "centroid": { "lat": 53.793, "long": 12.204 },
  "geometry_type": "point"
}
```

**Polygon objects** (nature reserves, flood zones, protected areas):
```json
"shape": {
  "centroid": { "lat": 53.15, "long": 10.23 },
  "bounds": {
    "top_left": { "lat": 53.20, "long": 10.15 },
    "bottom_right": { "lat": 53.10, "long": 10.30 }
  },
  "geometry_type": "polygon",
  "outline": {
    "type": "Polygon",
    "coordinates": [[[10.15, 53.20], [10.30, 53.20], ...]]
  }
}
```

Polygon objects often include a `wms` reference for rendering:

```json
"wms": {
  "url": "https://www.umweltkarten-niedersachsen.de/arcgis/services/Natur_wms/MapServer/WMSServer?",
  "type": "wms",
  "layer": {
    "name": "Naturschutzgebiet",
    "title": "Naturschutzgebiete",
    "legend_url": "https://...NSG.png"
  }
}
```

### Example: Nature reserve detail

```bash
curl 'https://api.umweltnavi.info/web/v1/ni/objects/nlwkn-nsg-nsg-lue-00363' \
  -H 'Origin: https://umweltportal.niedersachsen.de' \
  -H 'Accept: application/json'
```

Returns the "Oberes Lopautal" nature reserve with full description, photos, WMS layer for map rendering, GeoJSON polygon outline, data license, and links to related editorial pages.

---

## 8. Search — Full-Text Lookup

```
GET /{state}/search?q={query}
```

Searches across **all** objects and editorial pages by name and keyword.

### Parameters

| Parameter | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| `q` | string | min 3 chars, must contain a letter | Search query |

### Response

```json
{
  "results": [
    {
      "content": {
        "name": "Bruchbach",
        "category": "landschaftsschutzgebiet",
        "selectcategory": "landschaftsschutzgebiete"
      },
      "type": "object",
      "tags": ["spatial-object"],
      "ref": "nlwkn-lsg-lsg-ce-00035"
    },
    {
      "content": {
        "name": "Wildtier des Jahres"
      },
      "type": "page",
      "tags": [],
      "ref": 129
    }
  ]
}
```

### Two Result Types

| `type` | `ref` field | How to fetch detail |
|--------|-------------|---------------------|
| `"object"` | String (object ID) | `GET /{state}/objects/{ref}` |
| `"page"` | Integer (page ID) | `GET /{state}/pages/{ref}` |

### Example

```bash
# Search for kingfisher-related content
curl 'https://api.umweltnavi.info/web/v1/ni/search?q=eisvogel' \
  -H 'Origin: https://umweltportal.niedersachsen.de' \
  -H 'Accept: application/json'
```

Returns 74 results — nature reserves, landscape protection areas, and habitats where the kingfisher (Eisvogel) has been documented.

---

## 9. Pages — Editorial Content

```
GET /{state}/pages/{pageId}
```

Returns a CMS content page. Pages are referenced throughout the API — in subtopics, selectcategories, and object details — via `pages` arrays containing integer IDs.

### Response

```json
{
  "id": 81,
  "name": "Nordische Gastvögel",
  "teaser_text": "Im Winter kommen Millionen nordischer Gastvögel...",
  "teaser_image": {
    "url": "https://api.umweltnavi.info/v2/i/....jpg",
    "title": "Nonnengänse",
    "source": { "name": "NLWKN" },
    "license": { "name": "CC BY-SA 4.0" }
  },
  "info_text": "Detailed editorial text...",
  "content": [
    {
      "type": "text",
      "title": "Warum kommen sie?",
      "content": "<p>HTML-formatted editorial content...</p>"
    },
    {
      "type": "image",
      "url": "https://api.umweltnavi.info/v2/i/....jpg",
      "title": "Ringelgänse im Wattenmeer",
      "description": "Photo caption text",
      "source": { "name": "NLWKN" },
      "license": { "name": "CC BY 4.0", "url": "https://..." }
    },
    {
      "type": "link",
      "title": "Mehr erfahren",
      "url": "https://www.nlwkn.niedersachsen.de/..."
    }
  ]
}
```

### Content Block Types

The `content` array contains rich content blocks:

| `type` | Fields | Description |
|--------|--------|-------------|
| `text` | `title`, `content` | HTML-formatted text section |
| `image` | `url`, `title`, `description`, `source`, `license`, `author` | Image with attribution |
| `link` | `title`, `url` | External link |

### How to Find Page IDs

Page IDs appear in `pages` arrays on:
- **Subtopics** — editorial background for a topic section
- **Selectcategories** — information about a map layer
- **Object details** — related reading for a specific feature

Example: The "Oberes Lopautal" nature reserve references pages `[69, 34, 40, 53, 10]`, which contain editorial content about nature reserves, hiking trails, and local species.

---

## 10. Events — Press Releases & News

```
GET /{state}/events
```

Returns environmental press releases and news events. Events link to spatial objects and selectcategories, connecting news stories to the map.

### Response

```json
{
  "events": [
    {
      "id": "umweltnavi-c2a487c5",
      "type": "press_release",
      "title": "7,7 Millionen für länderübergreifendes Auenprojekt an der Unteren Wümme",
      "teaser_text": "Offizieller Start des Renaturierungsprojekts...",
      "info": "<p>Full HTML article content...</p>",
      "image": {
        "url": "https://api.umweltnavi.info/v2/i/....jpg",
        "title": "Wümme",
        "source": { "name": "Pixabay" },
        "license": { "name": "Inhaltslizenz" }
      },
      "publish_date": "2024-06-25T12:00:00+02:00",
      "end_date": "2024-08-27T12:00:00+02:00",
      "links": [
        { "id": "bfg-watercourse-de-494-wuemme", "type": "object" },
        { "id": "fliessgewaesser", "type": "selectcategory" },
        { "url": "https://www.umwelt.niedersachsen.de/...", "type": "link" }
      ],
      "selectcategories": ["fliessgewaesser", "gewaesserlauf"],
      "tags": ["event"]
    }
  ]
}
```

### Event Links

Each event has a `links` array connecting it to the portal:

| Link `type` | `id` / `url` field | How to use |
|-------------|---------------------|------------|
| `object` | Object ID string | Fetch with `GET /{state}/objects/{id}` — highlights feature on map |
| `selectcategory` | Selectcategory ID | Activates the corresponding map layer |
| `subtopic` | Subtopic ID | Links to a subtopic section |
| `link` | External URL | Opens in browser (usually a government press release) |

---

## 11. Data Sources & Object ID Prefixes

Object IDs encode their data source as a prefix. This tells you where the data comes from:

| Prefix | Source | Data Type |
|--------|--------|-----------|
| `oorg-` | [observation.org](https://observation.org) | Species sightings (dragonflies, birds, mammals, etc.) |
| `kwf-` | KWF (Kuratorium für Waldarbeit) | Forest rescue points (Rettungspunkte) |
| `mastr-` | [Marktstammdatenregister](https://www.marktstammdatenregister.de/) | Energy installations (wind, solar, biomass) |
| `bkg-` | Bundesamt für Kartographie und Geodäsie | Administrative boundaries (municipalities, counties) |
| `nlwkn-` | NLWKN (Niedersächsischer Landesbetrieb) | Protected areas (NSG, LSG, FFH, Natura 2000) |
| `numis-` | NUMIS (Nds. Umweltinformationssystem) | INSPIRE/OpenData metadata records |
| `bfg-` | Bundesanstalt für Gewässerkunde | Water bodies (rivers, lakes) |
| `gie-` | GIE (Gas Infrastructure Europe) | Gas storage facilities |

### Example IDs

```
oorg-libellen-363663219          → Dragonfly sighting from observation.org
kwf-rettungspunkte-a-s-5005     → Forest rescue point
mastr-ewind-see997219622973      → Offshore wind turbine from Marktstammdatenregister
nlwkn-nsg-nsg-lue-00363          → Nature reserve "Oberes Lopautal"
bkg-vg250-gemeinden-03241001     → Municipality boundary
numis-metadaten2-3f959eb9-...    → INSPIRE metadata record
bfg-watercourse-de-494-wuemme    → River "Wümme"
```

---

## 12. Error Handling

### HTTP Status Codes

| Code | Meaning | Response Body |
|------|---------|---------------|
| `200` | Success | JSON response (may be `null` for invalid states — see quirks) |
| `400` | Bad Request | JSON `{"message": "..."}`, JSON `null`, or HTML (from nginx) |
| `404` | Not Found | Empty (object or page does not exist) |
| `502` | Bad Gateway | HTML from nginx (backend timeout or crash) |

### 400 Error Examples

**Invalid coordinates:**
```json
{ "message": "Point {\"lat\":53.7841,\"long\":181} has invalid coordinates." }
```

**Elasticsearch query parse failure (malformed tag):**
```json
{ "message": "search_phase_execution_exception\n\tRoot causes:\n\t\tquery_shard_exception: Failed to parse query..." }
```

**Invalid state (nginx-level):**
```html
<html><head><title>400 Bad Request</title></head>...</html>
```

---

## 13. Known Limitations & API Quirks

These are behaviors discovered through automated fuzz testing with [Schemathesis](https://github.com/schemathesis/schemathesis) that users should be aware of:

### Invalid state returns `200` with `null` body

Sending an unsupported state code (anything other than `ni`, `sh`, `rp`) returns HTTP `200` with a `null` JSON body instead of `400`. Always check for `null` responses.

```bash
# Returns 200 with body: null
curl 'https://api.umweltnavi.info/web/v1/xx/topics' ...
```

### Large bounding boxes cause timeouts

Requesting objects across a very large area (e.g. spanning multiple countries) may cause the backend to time out, returning a `502 Bad Gateway` or hanging until connection timeout.

**Recommendation:** Keep bounding boxes reasonably small. At zoom level 16, a few kilometers is ideal.

### Longitude exactly ±180 crashes the backend

Setting longitude to exactly `180` or `-180` triggers an Elasticsearch tile calculation error. Use values strictly between -180 and 180.

### Search requires at least 3 characters with a letter

- Queries shorter than 3 characters are rejected with `400`
- Purely numeric queries (e.g. `"123"`) are rejected with `400`
- At least one alphabetic character (including umlauts) is required

### Elasticsearch error messages leak through

Some 400 and 200 responses contain raw Elasticsearch error messages (e.g. `search_phase_execution_exception`). These are backend errors that surface through the API rather than being wrapped in user-friendly messages.

---

## Appendix: Common Workflows

### Workflow A: Build a map layer

1. **Fetch topics** → `GET /ni/topics`
2. **Pick a selectcategory** from the tree (e.g. `libellen`)
3. **Query objects** → `GET /ni/objects/16/{lat1}/{lon1}/{lat2}/{lon2}?selectcategory=libellen`
4. **Render markers** using `shape.centroid` from each object
5. **On click** → `GET /ni/objects/{id}` for the full detail popup

### Workflow B: Search and display

1. **Search** → `GET /ni/search?q=eisvogel`
2. **For each result**, check `type`:
   - `"object"` → fetch `GET /ni/objects/{ref}` for map coordinates and detail
   - `"page"` → fetch `GET /ni/pages/{ref}` for editorial content
3. **Display** the object on a map or render the page content

### Workflow C: Show news on the map

1. **Fetch events** → `GET /ni/events`
2. **For each event link** with `type: "object"`:
   - Fetch `GET /ni/objects/{link.id}` to get the geographic location
3. **Plot** the news stories as markers on the map
4. **Show** the event's `teaser_text` and `image` in a popup

---

*This documentation was generated by analyzing the live API and validating the OpenAPI specification with automated fuzz testing (Schemathesis). The companion OpenAPI 3.0.3 spec (`umweltnavi-topics-api.yaml`) can be used with Swagger UI, code generators, or API testing tools.*
