# General

searx is a [metasearch-engine](https://en.wikipedia.org/wiki/Metasearch_engine), so it is using different search engines to provide better results. 

Because there is no general search-api which can be used for every search-engine, there must be build an adapter between searx and the external search-engine. This adapters are stored in the folder [_searx/engines_](https://github.com/asciimoo/searx/tree/master/searx/engines), and this site is build to make an general documentation about this engines

# general engine configuration

It is required to tell searx what results can the engine provide. The arguments can be inserted in the engine file, or in the settings file (normally ```settings.yml```). The arguments in the settings file override the one in the engine file.

Really, it is for most options no difference if there are contained in the engine-file or in the settings. But there is a standard where to place specific arguments by default.

### engine-file

| argument           | type     | information  |
| ------------------ | -------- | ------------ |
| categories         | list     | pages, in which the engine is working |
| paging             | boolean  | support multible pages |
| language_support   | boolean  | support language choosing |

### settings.yml

| argument           | type     | information  |
| ------------------ | -------- | ------------ |
| name               | string   | name of search-engine |
| engine             | string   | name of searx-engine (filename without .py) |
| shortcut           | string   | shortcut of search-engine |
| timeout            | string   | specific timeout for search-engine |

### overrides

There are some options, with have default values in the engine, but are often overwritten by the settings. If the option is assigned in the engine-file with ```None``` it has to be redefined in the settings, otherwise searx is not starting with that engine.

The naming of that overrides can be wathever you want. But we recommend the using of already used overrides if possible:

| argument           | type     | information  |
| ------------------ | -------- | ------------ |
| base_url           | string   | base-url, can be overwrite to use same engine on other url |
| number_of_results  | int      | maximum number of results per request |
| locale             | string   | _(unknow using)_ |
| api_key            | string   | api-key if required by engine |

### example-code

```python
# engine dependent config
categories = ['general']
paging = True
language_support = True
```

# doing request

To perform a search you have to specific at least a url on which the request is performing


### passed arguments

This arguments can be used to calculate the search-query. Furthermore, some of that parameters are filled with default values which can be changed for special purpose.

| argument           | type     | default-value, informations  |
| ------------------ | -------- | ---------------------------- |
| url                | string   | ```''``` |
| method             | string   | ```'GET'``` |
| headers            | set      | ```{}``` |
| data               | set      | ```{}``` |
| cookies            | set      | ```{}``` |
| verify             | boolean  | ```True``` |
| headers.User-Agent | string   | a random User-Agent |
| category           | string   | current category, like ```'general'```|
| started            | datetime | current date-time |
| pageno             | int      | current pagenumber |
| language           | string   | specific language code like ```'en_US'```, or ```'all'``` if unspecified |

### parsed arguments

The function ```def request(query, params):``` is always returning the ```params``` variable back. Inside searx, the following paramters can be used to specific a search-request:

| argument           | type     | information  |
| ------------------ | -------- | ------------ |
| url                | string   | requested url |
| method             | string   | HTTP request methode |
| headers            | set      | HTTP header informations |
| data               | set      | HTTP data informations (parsed if ```method != 'GET'```) |
| cookies            | set      | HTTP cookies |
| verify             | boolean  | Performing SSL-Validity check |

### example-code

```python
# search-url
base_url = 'https://example.com/'
search_string = 'search?{query}&page={page}'

# do search-request
def request(query, params):
    search_path = search_string.format(
        query=urlencode({'q': query}),
        page=params['pageno'])

    params['url'] = base_url + search_path

    return params
```

# returning results

Searx has the possiblity to return results in different media-types. Currently the following media-types are supported:

* default
* images
* videos
* torrent
* map

to set another media-type as default, you must set the parameter ```template``` to the required type.

### default

| result-parameter | information  |
| ---------------- | ------------ |
| url              | string, which is representing the url of the result |
| title            | string, which is representing the title of the result |
| content          | string, which is giving a general result-text |
| publishedDate    | [datetime.datetime](https://docs.python.org/2/library/datetime.html#datetime-objects), represent when the result is published  |

### images

to use this template, the parameter 

| result-parameter | information  |
| ---------------- | ------------ |
| template         | is set to ```images.html``` |
| url              | string, which is representing the url to the result site |
| title            | string, which is representing the title of the result _(partly implemented)_ |
| content          | _(partly implemented)_ |
| publishedDate    | [datetime.datetime](https://docs.python.org/2/library/datetime.html#datetime-objects), represent when the result is published _(not implemented yet)_ |
| img_src          | string, which is representing the url to the result image |
| thumbnail        | string, which is representing the url to a small-preview image _(not implemented yet)_ |

### videos

| result-parameter | information  |
| ---------------- | ------------ |
| template         | is set to ```videos.html``` |
| url              | string, which is representing the url of the result |
| title            | string, which is representing the title of the result |
| content          | _(not implemented yet)_ |
| publishedDate    | [datetime.datetime](https://docs.python.org/2/library/datetime.html#datetime-objects), represent when the result is published |
| thumbnail        |  string, which is representing the url to a small-preview image |

### torrent

| result-parameter | information  |
| ---------------- | ------------ |
| template         | is set to ```torrent.html``` |
| url              | string, which is representing the url of the result |
| title            | string, which is representing the title of the result |
| content          | string, which is giving a general result-text |
| publishedDate    | [datetime.datetime](https://docs.python.org/2/library/datetime.html#datetime-objects), represent when the result is published _(not implemented yet)_ |
| seed             | int, number of seeder |
| leech            | int, number of leecher |
| filesize         | int, size of file in bytes |
| files            | int, number of files |
| magnetlink       | string, which is the [magnetlink](https://en.wikipedia.org/wiki/Magnet_URI_scheme) of the result | 
| torrentfile      | string, which is the torrentfile of the result |
### map

| result-parameter | information  |
| ---------------- | ------------ |
| url              | string, which is representing the url of the result |
| title            | string, which is representing the title of the result |
| content          | string, which is giving a general result-text |
| publishedDate    | [datetime.datetime](https://docs.python.org/2/library/datetime.html#datetime-objects), represent when the result is published  |
| latitude         | latitude of result (in decimal format) |
| longitude        | longitude of result (in decimal format) |
| boundingbox      | boundingbox of result (array of 4. values ```[lat-min, lat-max, lon-min, lon-max]```) |
| geojson          | geojson of result (http://geojson.org) |
| osm.type         | type of osm-object (if OSM-Result) |
| osm.id           | id of osm-object (if OSM-Result) |
| address.name           | name of object |
| address.road           | street adress of object |
| address.house_number   | house number of object |
| address.locality       | city, place of object |
| address.postcode       | postcode of object |
| address.country        | country of object |