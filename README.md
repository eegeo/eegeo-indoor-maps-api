| WRLD Indoor Map Format |   | WRLD 3D indoor map |
|:-----------:|:-:|:------------:|
|![Indoor map source data](/images/tutorial/overview_coloured.png)|![Right arrow](/images/readme/arrow_right.png)|![Indoor map in app](/images/tutorial/overview.png)|

WRLD Indoor Map REST API v1.0.0
===============================

This repository contains details of the Open [CC-BY-4.0](https://github.com/wrld3d/wrld-indoor-maps-api/blob/master/LICENSE.md) WRLD Indoor Map Format &amp; API. 

---

Quick Start
===========

Get started creating 3D indoor maps with the [TUTORIAL.md](TUTORIAL.md)

* [TUTORIAL.md](TUTORIAL.md) - a worked example showing how to go from a floor plan to a 3D map
* [FORMAT.md](FORMAT.md) - a description of the Open GeoJSON-based WRLD Indoor Map Format
* [Full Indoor Map API Specification](#full-indoor-map-api-specification) - a description of the REST API for submitting maps in the WRLD Indoor Map Format
* [examples/](examples/) - a folder containing example indoor maps in the WRLD Indoor Map Format, including the correct directory structure to use when submitting indoor maps.
* [TROUBLESHOOTING.md](TROUBLESHOOTING.md) - tips for solving common problems


---

Full Indoor Map API Specification
=================================

## Indoor Maps

### Claim the location for which you will upload a new indoor map:

```sh
$ curl -v -XPOST https://indoor-maps-api.wrld3d.com/v1/edits/?token=<dev_auth_token> -F name="<venue_name>" -F venue_street_address="<address>" -F venue_phone_number="<phone no.>" -F venue_email="<email address>" -F submission_contact_email="<email address for notifications>" -F venue_outline="@</path/to/my/file>"
```

This will return a UUID string which can be used to identify the indoor map edit and query its status.

The supplied outline is a [GeoJSON](http://geojson.org/) file containing one or more polygons in the default [Coordinate Reference System](http://geojson.org/geojson-spec.html#coordinate-reference-system-objects).  This can be created via [QGIS](TUTORIAL.md), using online editors such as [GeoJSON.io](http://geojson.io) or with any other GIS software capable of exporting to GeoJSON.  

When you come to upload your indoor map, your edits will be checked against the GeoJSON submitted during this claim request, so please make sure the submitted polygons fully cover the regions you want to edit, but only intersect those buildings which form part of your indoor map.

### Query the status of an indoor map edit:

```sh
$ curl -v https://indoor-maps-api.wrld3d.com/v1/edits/UUID/status?token=dev_auth_token
```

### Upload a file against a new indoor map edit:

```sh
$ curl -v -XPUT https://indoor-maps-api.wrld3d.com/v1/edits/UUID?token=dev_auth_token -F comment="my venue comment" -F file="@/path/to/my/file"
```

This submits the map for processing, so the referenced file should contain the full detail that the user wants to see in the client.

### Query an indoor map edit:

```sh
$ curl -v https://indoor-maps-api.wrld3d.com/v1/edits/UUID?token=dev_auth_token
```

### Query all indoor map edits made by a developer 

```sh
$ curl -v https://indoor-maps-api.wrld3d.com/v1/edits/?token=dev_auth_token
```

### Download the claim outline for an indoor map

```sh
$ curl https://indoor-maps-api.wrld3d.com/v1.1/indoor-maps/UUID/claim?token=dev_auth_token > claim.geojson
```

### Delete an indoor map edit:

```sh
$ curl -v -XDELETE https://indoor-maps-api.wrld3d.com/v1/edits/UUID?token=dev_auth_token
```

### Update indoor data privacy setting

```sh
curl -v -X PUT https://indoor-maps-api.wrld3d.com/v1/indoor-data/UUID?token=dev_auth_token -d '{"private":true}'
```


### Give an API key access to a private indoors

```sh
curl -v -X POST https://indoor-maps-api.wrld3d.com/v1/api-keys/UUID?token=dev_auth_token -d '{"apikey":"<api_key>"}'
```

### Remove an API key access to a private indoors
```sh
curl -v -X DELETE https://indoor-maps-api.wrld3d.com/v1/api-keys/UUID/<api_key>?token=dev_auth_token
```

### Add custom user data

```sh
curl -v -X PUT https://indoor-maps-api.wrld3d.com/v1/indoor-data/UUID?token=dev_auth_token -d '{"user_data":{"key":"value"}}'
```

### Generate POI set from interior map

```sh
curl -v -XPOST https://indoor-maps-api.wrld3d.com/v1/poi/UUID?token=dev_auth_token -F feature_to_poi_key="@/path/to/my/file" -F "contact_email=example@mail.com"
```

#### Details for feature-to-poi key:
* **defaults:** Which interior feature types should have POIs generated, and what tag should be used to represent them.
* **special:** Which interior IDs should have POIs generated, and what tag should be used to represent them.
* **highlight_color:** What colour and opacity should be used for area highlights, where features have highlight set.

```json
{
  "defaults": {
    "interior feature type": "point of interest tag",
    "interior feature type": "point of interest tag",
    "interior feature type": "point of interest tag"
  },
  "special": {
    "point of interest tag": ["feature ID", "feature ID", "feature ID"],
    "point of interest tag": ["feature ID", "feature ID", "feature ID"],
    "point of interest tag": ["feature ID", "feature ID", "feature ID"]
  },
  "highlight_color": {
    "red": "0-255 red value",
    "green": "0-255 green value",
    "blue": "0-255 blue value",
    "opacity": "0-255 opacity value"
  }
}
```

#### Sample feature-to-poi key:
```json
{
  "defaults": {
    "room": "cocktail",
    "highlight": "hospital"
  },
  "special": {
    "entertainment":[318, 21, 400],
    "camping":[21, 52]
  },
  "highlight_color": {
    "red":0,
    "green":220,
    "blue":25,
    "opacity":128
  }
}
```

---

## Indoor Assets

### Create a new Indoor Asset set for a given Indoor Map and Floor

```sh
curl -v -XPOST https://indoor-maps-api.wrld3d.com/v1.1/indoor-maps/<indoor_uuid>/<floor_id>/entity-sets?token=<dev_auth_token> -F "name=<set_name>"
```

### Add a new Indoor Asset to a set

```sh
curl -v -X POST https://indoor-maps-api.wrld3d.com/v1.1/indoor-maps/<indoor_uuid>/<floor_id>/entity-sets/<set_id>?token=<dev_auth_token> -F "name=<asset_name>" -F "lat=<lat>" -F "lon=<lon>" -F "orientation=<orientation_in_degrees>" -F "model=<model_id>"
```
The `model` parameter should match a `prop_id` from our [prop manifest](https://cdn-resources.wrld3d.com/props/latest/Assets/manifest.json).

### Update an Indoor Asset

```sh
curl -v -X PUT https://indoor-maps-api.wrld3d.com/v1.1/indoor-maps/<indoor_uuid>/<floor_id>/entity-sets/<set_id>/entities/<entity_id>?token=<dev_auth_token> -F "name=<updated_name>" -F "orientation=<orientation_in_degrees>"
```

### Remove an Indoor Asset

```sh
curl -v -X DELETE https://indoor-maps-api.wrld3d.com/v1.1/indoor-maps/<indoor_uuid>/<floor_id>/entity-sets/<set_id>/entities/<entity_id>?token=<dev_auth_token>
```

### Bulk process Indoor Assets for a set

Similar to the POI API you can perform bulk operations that can create, update or delete existing entities in a set.

```sh
curl -v -X POST https://indoor-maps-api.wrld3d.com/v1.1/indoor-maps/<indoor_uuid>/<floor_id>/entity-sets/<set_id>/bulk?token=<dev_auth_token> -d '{
      "create":[
        {"name":"new", "lat":11.1, "lon":22.2, "orientation":45.0, "model":"circular_table_01"},
        {"name":"new2", "lat":33.3, "lon":44.4, "orientation":22.5, "model":"circular_table_01"}
      ],
      "update":[
        {"id":5, "name":"updated_name"}
      ],
      "delete":[
        6,
        7
      ]
    }'
```

### Delete an Indoor Asset Set

```sh
curl -v -X DELETE https://indoor-maps-api.wrld3d.com/v1.1/indoor-maps/<indoor_uuid>/<floor_id>/entity-sets/<set_id>?token=<dev_auth_token>
```

### Publish an Indoor Asset Set

```sh
curl -v -XPOST https://indoor-maps-api.wrld3d.com/v1.1/indoor-maps/<indoor_uuid>/<floor_id>/entity-sets/<set_id>/publish?token=<dev_auth_token>
```

For more information on publishing an Indoor Asset Set, see the [Indoor Assets Tutorial](TUTORIAL-ASSETS.md#publishing-your-assets).

### Download an Indoor Asset Set as a GeoJSON

```sh
curl -v https://indoor-maps-api.wrld3d.com/v1.1/indoor-maps/<indoor_uuid>/<floor_id>/entity-sets/<set_id>/entities.geojson?token=<dev_auth_token> > ./entities.geojson
```

### Importing Indoor Asset Sets

For more information on how to prepare an Indoor Asset AutoCAD DXF submission, refer to the [Indoor Assets Tutorial](TUTORIAL-ASSETS.md#preparing-an-asset-autocad-dxf-submission).

#### Import an Indoor Asset Set from an AutoCAD DXF

```sh
curl -v -XPOST "https://indoor-maps-api.wrld3d.com/v1.1/indoor-maps/<indoor_map_uuid>/<floor_id>/cad_conversions?token=<dev_auth_token>" -F 'file=@<path/to/file.zip>' -F 'submission_contact_email=<optional_email_address>'
```

#### Fetching the status of an Import Request

Calling the above endpoint submits a file to be processed by our pipeline that takes a short time to complete.  If you specified an email address, you will be mailed the result (success or failure) to that address upon completion.  If not you can fetch the current status via:

```sh
curl -v "https://indoor-maps-api.wrld3d.com/v1.1/indoor-maps/<indoor_map_uuid>/<floor_id>/cad_conversions/<job_id>?token=<dev_auth_token>"
```

The `status` parameter has the following values:

|Status|Value|Description|
 --- | --- | ---
|Waiting    | 0 | The conversion job is waiting in a queue to process.
|Working    | 1 | The conversion job is currently being processed.
|Failed     | 2 | The conversion job has failed to process.
|Success    | 3 | The conversion job has succeeded.

#### Fetching the GeoJSON result of an Import Request

```sh
curl -v "https://indoor-maps-api.wrld3d.com/v1.1/indoor-maps/<indoor_map_uuid>/<floor_id>/cad_conversions/<job_id>/result.geojson?token=<dev_auth_token>"
```

#### Getting more details on an Import Request Success or Failure

To fetch some additional information such as Warnings or Failures on a completed import request, you can call the following:

```sh
curl -v "https://indoor-maps-api.wrld3d.com/v1.1/indoor-maps/<indoor_map_uuid>/<floor_id>/cad_conversions/<job_id>/report.json?token=<dev_auth_token>"
```

#### Cancelling an Import Request

```sh
curl -v -XDELETE https://indoor-maps-api.wrld3d.com/v1.1/indoor-maps/<indoor_map_uuid>/<floor_id>/cad_conversions/<job_id>?token=<dev_auth_token>
```

---

License
===========

The WRLD Indoor Map data format is an open format released under Creative Commons Attribution 4.0 International. See the [LICENSE.md](https://github.com/wrld3d/wrld-indoor-maps-api/blob/master/LICENSE.md) file for details.

---

## Disclaimer
This is a stable, semantically versioned API. 

WRLD may make changes to the API from time to time but will adhere to [Semantic Versioning](http://semver.org/) and provide backward compatible end points for as long as possible.

---

## Contact us
If you have any problems or queries please [raise an issue](https://github.com/wrld3d/wrld-indoor-maps-api/issues/new).
