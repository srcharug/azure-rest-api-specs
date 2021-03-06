# Readme Configuration Guide for Python SDK
This file describe how to configure readme files to make it available for Python SDK code generation.

## Common Configuration
Configure basic package information.

### Basic Information
Configure package title/description/tag.
~~~~
// file: readme.md

``` yaml
title: AppconfigurationConfigurationClient
description: Appconfiguration Configuration Client
openapi-type: arm
tag: package-2019-10-01             // refer to below tag description
```
~~~~

### tag
Tags are used to define what swagger files are used in specific client SDK. In Single-API client, only one tag can be used to generate SDK client.
A tag can contains a bunch of swagger files which are used to generate the SDK. 

The name of a tag should be in form of package-yyyy-mm-dd[-xxx], for example below tag names are available:
- package-2020-02-03
- package-2020-03-22-preview
- package-2020-05-03-only

while those are invalid names:
- 2020-03-04
- package-preview-2020-03-04

A tag can be configured like below:
~~~~
// file: readme.md


### Tag: package-2019-10-01

These settings apply only when `--tag=package-2020-05-01` is specified on the command line.

``` yaml $(tag) == 'package-2019-10-01'
input-file:
- Microsoft.AppConfiguration/stable/2019-10-01/appconfiguration.json
- Microsoft.AppConfiguration/stable/2019-10-01/xxxx.json
```
~~~~


### all-api-versions
All swagger files need to be added into the all-api-version section. For Example:

~~~~
// file: readme.md


## Multi-API/Profile support for AutoRest v3 generators 

AutoRest V3 generators require the use of `--tag=all-api-versions` to select api files.

This block is updated by an automatic script. Edits may be lost!

``` yaml $(tag) == 'all-api-versions' /* autogenerated */
# include the azure profile definitions from the standard location
require: $(this-folder)/../../../profiles/readme.md

# all the input files across all versions
input-file:
  - Microsoft.AppConfiguration/stable/2019-10-01/appconfiguration.json
  - Microsoft.AppConfiguration/stable/2019-10-01/xxxx.json
  - Microsoft.AppConfiguration/stable/2018-10-01/appconfiguration.json
  - ...
~~~~


## Swagger to SDK
To make python SDK can be generated from the tag, swagger-to-sdk need to be configured:

~~~
// file: readme.md

## Swagger to SDK

This section describes what SDK should be generated by the automatic system.
This is not used by Autorest itself.

``` yaml $(swagger-to-sdk)
swagger-to-sdk:
  - repo: azure-sdk-for-python                          // for track1 SDK
  - repo: azure-sdk-for-python-track2                   // for track2 SDK
  - ...


## Python

See configuration in [readme.python.md](./readme.python.md)
~~~

## Python Configuration
Python dedicated configurations are configured in readme.python.md.
A typical readme.python.md is like this:

~~~
// file: readme.python.md


## Python

These settings apply only when `--python` is specified on the command line.
Please also specify `--python-sdks-folder=<path to the root directory of your azure-sdk-for-python clone>`.
Use `--python-mode=update` if you already have a setup.py and just want to update the code itself.

``` yaml !$(track2)                         // For track1: basic Python package information 
python-mode: create
python:
  azure-arm: true
  license-header: MICROSOFT_MIT_NO_VERSION
  payload-flattening-threshold: 2
  namespace: azure.mgmt.appconfiguration
  package-name: azure-mgmt-appconfiguration
  package-version: 0.1.0
  clear-output-folder: true
```

``` yaml $(python-mode) == 'update' && !$(track2)
python:                                     
  no-namespace-folders: true
  output-folder: $(python-sdks-folder)/appconfiguration/azure-mgmt-appconfiguration/azure/mgmt/appconfiguration
```

``` yaml $(python-mode) == 'create' && !$(track2)
python:
  basic-setup-py: true
  output-folder: $(python-sdks-folder)/appconfiguration/azure-mgmt-appconfiguration
```

These settings apply only when `--track2` is specified on the command line.

``` yaml $(track2)                          // For track2: basic Python package information 
azure-arm: true
license-header: MICROSOFT_MIT_NO_VERSION
package-name: azure-mgmt-appconfiguration
no-namespace-folders: true
package-version: 0.1.0
```

``` yaml $(python-mode) == 'update' && $(track2)
no-namespace-folders: true
output-folder: $(python-sdks-folder)/appconfiguration/azure-mgmt-appconfiguration/azure/mgmt/appconfiguration
```

``` yaml $(python-mode) == 'create' && $(track2)
basic-setup-py: true
output-folder: $(python-sdks-folder)/appconfiguration/azure-mgmt-appconfiguration
```
~~~

## Multi-API
Multi-API SDK is a package that support multiple REST api-versions. With Python multi-api SDK, the end user can assign api-version for REST calls explicitly; nevertheless, they can also use the default api-version which is choosed as the latest stable api-version.

If a multi-api package is need to be released, 'batch' should be used in the readme.python.md.
Typical multi-api RPs are azure-mgmt-compute, azure-mgmt-network, azure-mgmt-storage, you cam find their readme files to have a reference for multi-api.

### batch
The batch is a tag list which are used in the multi-api package. For example:
~~~
// file: readme.python.md

### Python multi-api

Generate all API versions currently shipped for this package

```yaml $(multiapi) && !$(track2)           // For track1
batch:
  - tag: package-2020-06-01-only
  - tag: package-2020-05-01-only
```

```yaml $(multiapi) && $(track2)            // For track2
clear-output-folder: true
batch:
  - tag: package-2020-06-01-only
  - tag: package-2020-05-01-only
  - multiapiscript: true
```

``` yaml $(multiapiscript) && $(track2)      // For track2
output-folder: $(python-sdks-folder)/compute/azure-mgmt-compute/azure/mgmt/compute/
clear-output-folder: false
perform-load: false
```
~~~

### tags of batch
Then, the output folder and namespace should be configured for each of the tag. For example:
~~~
// file: readme.python.md

### Tag: package-2020-06-01-only and python

These settings apply only when `--tag=package-2020-06-01-only --python` is specified on the command line.
Please also specify `--python-sdks-folder=<path to the root directory of your azure-sdk-for-python clone>`.

``` yaml $(tag) == 'package-2020-06-01-only' && $(track2)      // For track2
namespace: azure.mgmt.compute.v2020_06_01
output-folder: $(python-sdks-folder)/compute/azure-mgmt-compute/azure/mgmt/compute/v2020_06_01
```
~~~


### multi-api init script
For track1, multi-api init script need to be configured in readme.md. For example:

~~~
// file: readme.md

swagger-to-sdk:
  - repo: azure-sdk-for-python
    after_scripts:
      - python ./scripts/multiapi_init_gen.py azure-mgmt-compute
~~~


## Run codegen
After configure all the readme files, autorest can be used to generate SDK.

### Track1 (for Autorest V2)
Track1 SDK is based on AutoRest version V2 that's going to be replaced by version V3.

~~~
> autorest --keep-version-file --multiapi --no-async --python --python-mode=update --python-sdks-folder=C:\ZZ\projects\codegen\azure-sdk-for-python\sdk --use=@microsoft.azure/autorest.python@~4.0.71 --version=V2 ..\azure-rest-api-specs\specification\appconfiguration\resource-manager\readme.md
~~~

### Track2 (for latest Autorest)
Track 2 is based on the latest AutoRest code generator

~~~
> autorest --reset
> autorest --python --track2 --use=@autorest/python@5.4.1 --python-sdks-folder=..\azure-sdk-for-python\sdk --multiapi --python-mode=update  ..\azure-rest-api-specs\specification\appconfiguration\resource-manager\readme.md
~~~