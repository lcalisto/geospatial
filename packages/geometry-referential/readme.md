# **Geometry Referential**

A package to convert coordinates; formats and altitudes.

---

## What is referential package?

**Referential** is a package that can deal with **format** convertions; **coordinate** transformations and **altitude** convertions.

There are two main curried functions, a **syncronous** function: `convert(from,to)(coordinates)` and one **asyncronous** function: `convertAsync(from,to)(coordinates)` althought both functions (sync and async) have the same signature, **altitude** convertions are only performed when using the asyncronous function (convertAsync). We use curried functions to make it easier, and more eficient, to process big amonts of points. We can create one conversion and then apply it to several points, for example:

```typescript
const conversion1 = convert(from, to);

const coordinates1 = conversion1(coordinatesA);
const coordinates2 = conversion1(coordinatesB);
const coordinates3 = conversion1(coordinatesC);
const coordinates4 = conversion1(coordinatesD);
const coordinates5 = conversion1(coordinatesE);
```

---

## Format convertions

All supported formats can be transformed between them.

- ArrayXY
- ArrayXYZ
- ArrayLatitudeLongitude
- ArrayLatitudeLongitudeAltitude
- String e.g.
- ObjectXY
- ObjectXYZ
- ObjectLatLon
- ObjectLatLonAlt
- ObjectLatitudeLongitude
- ObjectLatitudeLongitudeAltitude
- ThreeVector2
- ThreeVector3
- ThreeVector4
- CesiumCartographic
- CesiumCartesian2
- CesiumCartesian3
- CesiumCartesian4

Please check [src/types.ts](src/types.ts) to see all supported formats.

We can transform a coordinate from array to object like:

```typescript
const myConvertion = convert(
  {
    format: {
      type: "arrayXYZ"
    }
  },
  {
    format: {
      type: "objectXYZ"
    }
  }
);
const result = myConvertion([-1.9425367, 46.68808, 0]);
```

Since convert is a curried function if you dont need to reuse this convertion in multiple objects you can just do:

```typescript
const result = convert(
  {
    format: {
      type: "arrayXYZ"
    }
  },
  {
    format: {
      type: "objectXYZ"
    }
  }
)([-1.9425367, 46.68808, 0]);
```

In both cases the result will be:

```typescript
{ x: -1.9425367, y: 46.68808, z: 0 }
```

Any format convertion ncan be performed with both SYNC and ASYNC functions. We can rewrite the previous example uing the async function as

```typescript
const myConvertion = await convertAsync(
  {
    format: {
      type: "arrayXYZ"
    }
  },
  {
    format: {
      type: "objectXYZ"
    }
  }
);

const result = await myConvertion([-1.9425367, 46.68808, 0]);
```

Its possible to use this functions with both Cesium and Three coordinate objects:

```typescript
import * as THREE from "three";
import cesium from "cesium";

const cesiumPointObject = new cesium.Cartographic(-1.9425367, 46.68808, 0);

const myConvertion = convert(
  {
    format: {
      type: "cesiumCartographic",
      cesium: cesium
    }
  },
  {
    format: {
      type: "threeVector3",
      three: THREE
    }
  }
);

const result = myConvertion(cesiumPointObject);
```

For this example the result should be an object like

```typescript
new THREE.Vector3(-1.9425367, 46.68808, 0);
```

**Warning:** Make sure you provide both _THREE_ and/or _Cesium_ instances in the format object. For this particular cases this is a mandatory field.

---

## Coordinate transformations

At this moment this package uses a modified version of proj4js (http://proj4js.org) available at: https://github.com/lcalisto/proj4js This is a normal proj4js with tha ability of transforming from and to geocentric coordinates. This means that most of coordinate transformations are possible.

One example of transforming from 4326 (WGS84) to 3763 (Portugal TM06) can be described as

```typescript
const myConvertion = convert(
  {
    system: {
      type: "reference",
      definition: "+proj=longlat +datum=WGS84 +no_defs",
      altitudeReference: "default"
    },
    format: {
      type: "arrayXYZ"
    }
  },
  {
    system: {
      type: "reference",
      definition:
        "+proj=tmerc +lat_0=39.66825833333333 +lon_0=-8.133108333333334 +k=1 +x_0=0 +y_0=0 +ellps=GRS80 +towgs84=0,0,0,0,0,0,0 +units=m +no_defs",
      altitudeReference: "default"
    },
    format: {
      type: "arrayXYZ"
    }
  }
);
const result = myConvertion([
  -7.56233996602137,
  38.96618001296369,
  2.9946155063807964
]);
```

In this example the result should be:

```typescript
[49467.31334521535, -77790.87043755408, 2.9946155063807964];
```

For coordinate transformations and altitude convertions we must always provide _system_ object

```json
    system: {
      type: "reference",
      definition:
        "+proj=tmerc +lat_0=39.66825833333333 +lon_0=-8.133108333333334 +k=1 +x_0=0 +y_0=0 +ellps=GRS80 +towgs84=0,0,0,0,0,0,0 +units=m +no_defs",
      altitudeReference: "default"
    }
```

All types are described in [src/types.ts](src/types.ts)

Allowed _type_ values are _reference_ or _local_ Referenced systems must have a definition which is a proj4 string definition. When using `convertAsync` we can provide EPSG code. Please note that this is only vcalid for the `convertAsync` function.

The folowing example transforms from WGS84 to EPSG:3763 (Portugal TM06) and change format from xyz (longitude latitude altitude) string to arrayXYZ using the Async function.

```typescript
const testingPointObject = "38°57'58\" E, 7°33'44\" S, 2.9946155045181513 m";
const myconvertion = await convertAsync(
  {
    system: {
      type: "reference",
      definition: "EPSG:4326",
      altitudeReference: "default"
    },
    format: {
      type: "string"
    }
  },
  {
    system: {
      type: "reference",
      definition: "EPSG:3763",
      altitudeReference: "default"
    },
    format: {
      type: "arrayXYZ"
    }
  }
);
const result = await myconvertion(testingPointObject);
```

In this example the result should be:

```typescript
[49477.565958628555, -77798.4557088422, 2.994615505449474];
```

### Local transformations

Its possible to transform from and to local coordinate systems. When dealing with local systems its mandatory to specify the origin relative to a reference system

The following example transforms a local coordinate system to WGS84

```typescript
const myConvertion = await convertAsync(
  {
    system: {
      type: "local",
      origin: {
        value: [-7.56233996602137, 38.96618001296369, 2.9946155063807964],
        formattedSystem: {
          system: {
            type: "reference",
            definition: "+proj=longlat +datum=WGS84 +no_defs",
            altitudeReference: "default"
          },
          format: {
            type: "arrayXYZ"
          }
        }
      }
    },
    format: {
      type: "arrayXYZ"
    }
  },
  {
    system: {
      type: "reference",
      definition: "+proj=longlat +datum=WGS84 +no_defs",
      altitudeReference: "default"
    },
    format: {
      type: "arrayXYZ"
    }
  }
);

const result = await result([0, 0, 0]);
```

In this example, because we are requesting the origin, the result should be:

```typescript
[-7.56233996602137, 38.96618001296369, 2.9946155063807964];
```

---

## Altitude conversions

Altitude conversions can **only** be processed when using `convertAsync` function.

Currently supported altitude types are:

- cesiumTerrain, uses Cesium library;
- geoid (egm96);
- default, uses ellipsoid (Proj4);

The folowing image describes diferences between the three diferent altitudes.

![alt text](https://i.stack.imgur.com/uY8aF.jpg "Altitudes")

The folowing example converts altitude from default to cesiumTerrain

```typescript
const myConvertion = await convertAsync(
  {
    system: {
      type: "reference",
      definition: "+proj=longlat +datum=WGS84 +no_defs",
      altitudeReference: "default"
    },
    format: {
      type: "arrayXYZ"
    }
  },
  {
    system: {
      type: "reference",
      definition: "+proj=longlat +datum=WGS84 +no_defs",
      altitudeReference: "cesiumTerrain"
    },
    format: {
      type: "arrayXYZ"
    }
  }
);
const result = await myConvertion([-7.49024, 38.41922, 0]);
```

In this example the result should be:

```typescript
[-7.49024, 38.41922, -228.4332850880927];
```

In this particular example altitude is negative because it's decribed relative to the terrain. In other words, for this particular possition terrain is 228.433 meters above ellipsoid.

---
