# tessera

tessera is a [tilelive](https://github.com/mapbox/tilelive.js)-based tile
server.

Using the power of the tilelive ecosystem, it is capable of serving and
rendering tiles from many sources.

## Installation

```bash
npm install -g tessera
```

## How to Use

To serve up an [MBTiles](https://www.mapbox.com/developers/mbtiles/) archive
using [node-mbtiles](https://github.com/mapbox/node-mbtiles):

```bash
tessera mbtiles://./whatever.mbtiles
```

To serve up a [TM2](https://github.com/mapbox/tm2) style using
[tilelive-tmstyle](https://github.com/mojodna/tilelive-tmstyle):

```bash
tessera tmstyle://./project.yml
```

**Note**: non-`mapbox:` sources may need to have their protocols changed;
tessera requires that styles using HTTP-accessible data have `tilejson+http:`
as their protocol where TM2 expects `http:`.  See
[mojodna/tilelive-http#2](https://github.com/mojodna/tilelive-http/issues/2)
for more information.

To serve up a [TM2](https://github.com/mapbox/tm2) data source (it will use
`data.yml` as the source of truth) using
[tilelive-tmsource](https://github.com/mojodna/tilelive-tmsource):

```bash
tessera tmsource://./
```

To serve up a bare [Mapnik](https://github.com/mapnik/mapnik) stylesheet using
[tilelive-mapnik](https://github.com/mapbox/tilelive-mapnik):

```bash
tessera mapnik://./stylesheet.xml
```

To serve up files from a filesystem using
[tilelive-file](https://github.com/mapbox/tilelive-file):

```bash
tessera file://./tiles
```

To proxy HTTP-accessible tiles using
[tilelive-http](https://github.com/mojodna/tilelive-http):

```bash
tessera http://tile.stamen.com/toner/{z}/{x}/{y}.png
```

To proxy Mapbox-hosted tiles using
[tilelive-mapbox](https://github.com/mojodna/tilelive-mapbox):

```bash
tessera mapbox:///mapbox.mapbox-streets-v4
```

To proxy tiles with available
[TileJSON](https://www.mapbox.com/developers/tilejson/) using
[node-tilejson](https://github.com/mapbox/node-tilejson):

```bash
tessera tilejson+http://a.tiles.mapbox.com/v3/mapbox.mapbox-streets-v4.json
```

A TileJSON endpoint is available at
[localhost:8080/index.json](http://localhost:8080/index.json) with various bits
of metadata about the tiles being served.

## Configuration

Tessera has command-line options:

```bash
Usage: node tessera.js [uri] [options]

uri     tilelive URI to serve

Options:
   -C SIZE, --cache-size SIZE   Set the cache size (in MB)  [10]
   -c CONFIG, --config CONFIG   Provide a configuration file
   -p PORT, --port PORT         Set the HTTP Port  [8080]
   -v, --version                Show version info

A tilelive URI or configuration file is required.
```

This is what a configuration file looks like:

```javascript
{
  "/": {
    "source": "mbtiles:///Users/seth/archive.mbtiles",
    "cors": false,
    "timing": false
  },
  "/a": {
    "source": "mbtiles:///Users/seth/archive.mbtiles",
    "headers": {
      "Cache-Control": "public,max-age={{#tileJSON}}86400{{/tileJSON}}{{#tile}}3600{{/tile}}",
      "Surrogate-Control": "max-age=86400",
      "Surrogate-Keys": "{{#tile}}z{{zoom}} x{{x}} y{{y}}{{/tile}}"
    }
  },
  "/b": "mbtiles:///Users/seth/archive.mbtiles"
}
```

Header values are treated as
[Mustache](http://mustache.github.io/mustache.5.html) (technically
[Handlebars](http://handlebarsjs.com/)) templates, which allow them to vary by
request. The following variables are available to header templates:

* `tile.retina` - for retina (`@2x`) requests
* `tile.zoom` - zoom (for tile requests)
* `tile.x` - row (for tile requests)
* `tile.y` - column (for tile requests)
* `tile.format` - requested format
* `tileJSON` - for TileJSON requests
* `200` - HTTP 200
* `404` - HTTP 404
* `invalidFormat` - the requested format did not match what the tilelive source
  provides
* `invalidZoom` - the requested zoom is outside the available range
* `invalidBounds` - the requested coordinates are outside the available bounds

CORS and `X-Response-Time` can be disabled per-style:

```javascript
{
  "cors": false,
  "timing": false
}
```

(Note that enabling for `/` will propagate to all subdirectories, as they act
as middleware.)

## Environment Variables

* `PORT` - Port to bind to. Defaults to `8080`.
* `CACHE_SIZE` - Cache size (in MB) for
  [tilelive-cache](https://github.com/mojodna/tilelive-cache). Defaults to
  10MB.
