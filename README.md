# NAME

y2j, j2y, yq - filters to convert JSON to YAML, YAML to JSON and YAML to YAML.

## SYNOPSIS

```bash
# convert from YAML to JSON
y2j < yaml > json
j2y -d < yaml > json

# convert from YAML to JSON with optional trailing jq transformation
y2j {jq-filter} < yaml

# convert from JSON to YAML
j2y < json > yaml
y2j -d < json > yaml

# convert from JSON to YAM with optional leading jq transformation
j2y {jq-filter} < json > yaml

# convert YAML to JSON, run jq, convert back to YAML
yq {jq-filter} < yaml > yaml

# create an installer that will install y2j.sh into /usr/local/bin, then run that script with bash
y2j.sh installer /usr/local/bin | sudo bash
```

## DESCRIPTION

This package provides filters for transforming JSON to YAML, YAML to JSON and YAML to YAML.

YAML to YAML transformations are performed by applying a jq filter to a JSON transformation
of the YAML input stream with y2j and transforming the resulting JSON stream back to YAML with j2y.

The script will use the local instances of jq, python and the required python modules if they exist locally
or will use a docker container based on the wildducktheories/y2j image otherwise.

## INSTALLATION

```bash
docker run --rm wildducktheories/y2j y2j.sh installer /usr/local/bin | sudo bash
```

Replace /usr/local/bin with a different directory to specify a different installation location or omit to
default to /usr/local/bin.

If the installer fails with complaints about lack of a running docker daemon or failure to find a wildducktheories/y2j image, consider changing ```sudo bash``` to ```sudo -E bash``` so that root inherits the current user's docker environment.

## EXAMPLES

### j2y

```bash
echo '{ "foo": [ { "id": 1 } , {"id": 2 }] }' | j2y
```

yields:

```yaml
foo:
- id: 1
- id: 2
```

### j2y - with jq pre-stage

```bash
echo '{"foo": "bar"}{"foo": "baz"}' | j2y -s .
```

yields:

```yaml
- foo: bar
- foo: baz
```

### y2j

```bash
(
    y2j <<EOF
foo:
- id: 1
- id: 2
EOF
) | jq -c .
```

yields:

```json
{"foo":[{"id":1},{"id":2}]}
```

### yq

```bash
yq '.foo=(.foo[]|select(.id == 1)|[.])' <<EOF
foo:
- id: 1
- id: 2
EOF
```

yields:

```yaml
foo:
- id: 1
```

## LIMITATIONS

* y2j and yq only support the subset of YAML streams that can be losslessly represented in JSON - that is: trees. Graphs, anchors and references are not supported.
* j2y only supports reading of a single JSON object or a single JSON array from stdin. If the JSON input contains
multiple objects, consider using '-s .' with j2y to slurp the input into a single JSON array.
* yq only supports jq-filters that are guaranteed to produce a single JSON object or array.

Behaviour with inputs or filters that do not satisfy these constraints is not defined.

## AUTHOR

Jon Seymour - jon@wildducktheories.com

## ACKNOWLEDGMENTS

Conversions used by y2j.sh are based on the commandlinefu scripts found here:

* [convert-yaml-to-json](https://www.commandlinefu.com/commands/view/12218/convert-yaml-to-json)
* [convert-json-to-yaml](https://www.commandlinefu.com/commands/view/12219/convert-json-to-yaml)

Filtering is implemented with jq. See [jq](https://stedolan.github.io/jq/).

## REVISIONS

### 1.2

* Fork by Graham Pugh - graham@grahamrpugh.com. Switches to using python3, and quits gracefully if python3 and docker are not installed.

### 1.1.1

* Fixed typo in installation message.
* Support running on Linux.

Both fixes thanks to reports submitted by Henrik Holmboe.

### 1.1

* Added support for yq.

### 1.0

* Initial release.