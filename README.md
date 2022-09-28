# ProcessLib API
## Background
For the new logo wizard and Bulk Design Maker features, we need to provide the functionality to render stages and variations.

https://breezi.assembla.com/spaces/Placeit-Agile/tickets/2256-placeit--new-logo-maker--logo-render--processlib-api/details

### Related documents:

[#PA-1596 - Placeit: New Logo Maker: Logo fast render: Frontend: Problem Discovery](https://docs.google.com/document/d/1HkEqgW0okGhlgQvEEVNK_dTHbd5XAbQct5PHA2BLyUQ/edit?usp=sharing)

[#PA-2005 - Placeit: New Logo Maker: Logo render: Frontend: Process](https://docs.google.com/document/d/1iZ6zHfxmaGI7WOLyk7SvjggD3A3sdzrCzrckWQ6JGBI/edit?usp=sharing)

[#PA-1981 - Placeit: New Logo Maker: Logo render: Frontend: Prepare data to be processed: Problem Discovery.](https://docs.google.com/document/d/1aYyjml7wZf2VU-g57FJWLIXMFImjbcRQJ9wk-fofDl8/edit)

## Problem
- We need to implement an API to establish the communication between the Client and the render Process Lib.
- We need to implement a mechanism so that the Client can cancel requests made to the render process API.

## Hypothesis
If we create an API to establish communication between the client and the process render, we’ll be able to have standards defined to request the render process.

## Solution Proposal
Define the API to establish the communication between the client and the process render service, with the following features:
- Request the render of a stage (base64 image) from the stage data and the values tagged.
- Request the render (base64 image) of the pack of stages from a set of stage data and the values tagged.
- Request the render (base64 image) of the pack of variations from the same stage data and a set of values tagged.
- The request should be cancellable.
- An optional “renderer” option should be available

## Theory of operation
- An optional “renderer” function, a “value” object and a “stagesToProcess” array serve as inputs.
- The structure, ui, preset and size json files will be fetched for each entry of the stagesToProcess array
    - The ui and preset json will be merged outputting a record
    - The changes defined in the Values object will be applied to the record
    - The record and json files will be fed to an instance of the ngin to be processed
    - A render process will happen in which a screenshot of the stage will be taken and returned as base64
- An array will be returned with cancellable promises for each stage, each will resolve in a base64 image.

## Black Box
[![Black Box](https://blog.mozilla.org/design/files/2019/06/Logo-Lockups.svg "Black Box")](https://bit.ly/3Rj4OhI)

## Flow Diagram
[![Flow Diagram](https://blog.mozilla.org/design/files/2019/06/FX_Design_Logo_Browser-logos-lockup-2.svg "Flow Diagram")](https://dreampuf.github.io/GraphvizOnline/#digraph%20G%20%7B%0A%20%20%20%20stage%5Blabel%3D%3CStage%3E%2C%20shape%3Dbox%5D%3B%0A%20%20%20%20value%5Blabel%3D%3CValue%3E%2C%20shape%3Dbox%5D%3B%0A%20%20%20%20function%5Blabel%3D%3CFunction%3E%2C%20shape%3Dbox%5D%0A%20%20%20%20promise3%5Blabel%3D%3CCancellable%20Promise%3Cbr%2F%3E---------%3Cbr%2F%3EResolves%20base64%20image%3E%2C%20shape%3Dbox%5D%3B%0A%0A%20%20%20%20subgraph%20cluster_2%20%7B%0A%20%20%20%20%20%20%20%20style%3Dfilled%3B%0A%20%20%20%20%20%20%20%20color%3Dblack%3B%0A%20%20%20%20%20%20%20%20fontcolor%3Dwhite%3B%0A%20%20%20%20%20%20%20%20label%20%3D%20%22renderStage%22%3B%0A%20%20%20%20%20%20%20%20node%20%5Bshape%3Dbox%2Cstyle%3Dfilled%2Ccolor%3Dwhite%5D%3B%0A%20%20%20%20%20%20%20%20prepareData3%5Blabel%3D%3CprepareData%3E%5D%3B%0A%20%20%20%20%20%20%20%20mergeChanges3%5Blabel%3D%3CmergeChanges%3E%5D%3B%0A%20%20%20%20%20%20%20%20renderProcess3%5Blabel%3D%3CrenderProcess%3E%5D%3B%0A%20%20%20%20%20%20%20%20%2F%2F%20singleStageProcessorBuilder%20-%3E%20singleStageProcessor%20%5Bcolor%3Dwhite%5D%3B%0A%20%20%20%20%20%20%20%20singleStageProcessor%20-%3E%20prepareData3%20-%3E%20mergeChanges3%20-%3E%20renderProcess3%20%5Bcolor%3Dwhite%5D%3B%0A%20%20%20%20%20%20%20%20%2F%2F%20%7Brank%3Dsame%3B%20singleStageProcessorBuilder%3B%20singleStageProcessor%3B%7D%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20function%20-%3E%20singleStageProcessor%20%5Bcolor%3D%22black%3B0.25%3Awhite%22%5D%3B%0A%20%20%20%20value%20-%3E%20singleStageProcessor%20%5Bcolor%3D%22black%3B0.25%3Awhite%22%5D%3B%0A%20%20%20%20stage%20-%3E%20singleStageProcessor%20%5Bcolor%3D%22black%3B0.25%3Awhite%22%5D%3B%0A%20%20%20%20singleStageProcessor%20-%3E%20promise3%20%5Bcolor%3D%22white%3B0.915%3Ablack%22%2C%20weight%3D%222%22%5D%3B%0A%20%20%20%20renderProcess3%20-%3E%20promise3%20%5Bstyle%3Ddashed%2C%20color%3D%22white%3B0.30%3Ablack%22%5D%3B%0A%0A%20%20%20%20stages%5Blabel%3D%3CstagesToProcess%3E%2C%20shape%3Dbox3d%5D%3B%0A%20%20%20%20values%5Blabel%3D%3CValue%3E%2C%20shape%3Dbox%5D%3B%0A%20%20%20%20function3%5Blabel%3D%3CFunction%3E%2C%20shape%3Dbox%5D%0A%20%20%20%20promise%5Blabel%3D%3CCancellable%20Promise%20Array%3Cbr%2F%3E---------%3Cbr%2F%3EResolves%20base64%20images%3E%2C%20shape%3Dbox%5D%3B%0A%0A%20%20%20%20subgraph%20cluster_0%20%7B%0A%20%20%20%20%20%20%20%20style%3Dfilled%3B%0A%20%20%20%20%20%20%20%20color%3Dblack%3B%0A%20%20%20%20%20%20%20%20fontcolor%3Dwhite%3B%0A%20%20%20%20%20%20%20%20label%20%3D%20%22renderStages%22%3B%0A%20%20%20%20%20%20%20%20node%20%5Bshape%3Dbox%2Cstyle%3Dfilled%2Ccolor%3Dwhite%5D%3B%0A%20%20%20%20%20%20%20%20%2F%2F%20multiStageProcessorBuilder%20-%3E%20multiStageProcessor%20%5Bcolor%3Dwhite%5D%3B%0A%20%20%20%20%20%20%20%20multiStageProcessor%20-%3E%20prepareData%20-%3E%20mergeChanges%20-%3E%20renderProcess%20%5Bcolor%3Dwhite%5D%3B%0A%20%20%20%20%20%20%20%20%2F%2F%20%7Brank%3Dsame%3B%20multiStageProcessorBuilder%3B%20multiStageProcessor%3B%7D%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20function3%20-%3E%20multiStageProcessor%20%5Bcolor%3D%22black%3B0.25%3Awhite%22%5D%3B%0A%20%20%20%20stages%20-%3E%20multiStageProcessor%20%5Bcolor%3D%22black%3B0.25%3Awhite%22%5D%3B%0A%20%20%20%20values%20-%3E%20multiStageProcessor%20%5Bcolor%3D%22black%3B0.25%3Awhite%22%5D%3B%0A%20%20%20%20multiStageProcessor%20-%3E%20promise%20%5Bcolor%3D%22white%3B0.915%3Ablack%22%2C%20weight%3D%222%22%5D%3B%0A%20%20%20%20renderProcess%20-%3E%20promise%20%5Bstyle%3Ddashed%2C%20color%3D%22white%3B0.30%3Ablack%22%5D%3B%0A%0A%20%20%20%20values2%5Blabel%3D%3CvaluesToProcess%3E%2C%20shape%3Dbox3d%5D%3B%0A%20%20%20%20stages2%5Blabel%3D%3CStage%3E%2C%20shape%3Dbox%5D%3B%0A%20%20%20%20function2%5Blabel%3D%3CFunction%3E%2C%20shape%3Dbox%5D%0A%20%20%20%20promise2%5Blabel%3D%3CCancellable%20Promise%20Array%3Cbr%2F%3E---------%3Cbr%2F%3EResolves%20base64%20images%3E%2C%20shape%3Dbox%5D%3B%0A%0A%20%20%20%20subgraph%20cluster_1%20%7B%0A%20%20%20%20%20%20%20%20style%3Dfilled%3B%0A%20%20%20%20%20%20%20%20color%3Dblack%3B%0A%20%20%20%20%20%20%20%20fontcolor%3Dwhite%3B%0A%20%20%20%20%20%20%20%20label%20%3D%20%22renderVariations%22%3B%0A%20%20%20%20%20%20%20%20node%20%5Bshape%3Dbox%2Cstyle%3Dfilled%2Ccolor%3Dwhite%5D%3B%0A%20%20%20%20%20%20%20%20prepareData2%5Blabel%3D%3CprepareData%3E%5D%3B%0A%20%20%20%20%20%20%20%20mergeChanges2%5Blabel%3D%3CmergeChanges%3E%5D%3B%0A%20%20%20%20%20%20%20%20renderProcess2%5Blabel%3D%3CrenderProcess%3E%5D%3B%0A%20%20%20%20%20%20%20%20variationsProcessor%5Blabel%3D%3CvariationsProcessor%3E%5D%3B%0A%20%20%20%20%20%20%20%20variationsProcessor%20-%3E%20prepareData2%20-%3E%20mergeChanges2%20-%3E%20renderProcess2%5Bcolor%3Dwhite%5D%3B%0A%20%20%20%20%20%20%20%20%2F%2F%20%7Brank%3Dsame%3B%20variationsProcessorBuilder%3B%20variationsProcessor%7D%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20function2%20-%3E%20variationsProcessor%20%5Bcolor%3D%22black%3B0.25%3Awhite%22%5D%3B%0A%20%20%20%20stages2%20-%3E%20variationsProcessor%20%5Bcolor%3D%22black%3B0.25%3Awhite%22%5D%3B%0A%20%20%20%20values2%20-%3E%20variationsProcessor%20%5Bcolor%3D%22black%3B0.25%3Awhite%22%5D%3B%0A%20%20%20%20renderProcess2%20-%3E%20promise2%20%5Bstyle%3Ddashed%2C%20color%3D%22white%3B0.35%3Ablack%22%5D%3B%0A%20%20%20%20variationsProcessor%20-%3E%20promise2%20%5Bcolor%3D%22white%3B0.915%3Ablack%22%2C%20weight%3D%222%22%5D%3B%0A%7D)

## Functional Specification
### Type
#### stageData : object
- Id: string
- presetID: string (optional)
- sizeId: string (optional)

#### value : object
- text1: object
    - color: string
    - contents: string
    - font: object
        - name: string
        - displayname: string
        - file: string
        - image: string
        - visible: boolean
        - isPremium: boolean
- backgroundcolor1: object
    - color: string
- graphic1: object
    - layer: object
        - assetType: string
        - id: string
        - name: string
        - type: string
        - tags: array
        - …………
        - originalFile: string
        - image: string
        - path: string
        - visible: boolean

### Method
<br>

#### `multiStageProcessor(renderer:function;optional)(value:object)(stageData:array) : Array(Cancellable Promise)`
Receives:
>|Name|Type|Description|
>|-|-|-|
>|renderer|Function|A custom render function that will resolve into a base64 image of a stage.<br>Said function will receive as arguments an object containing the structure and ui.|
>|value|Object|A [value](#value--object) object containing the user changes|
>|stageData|Array|An array of [stageData](#stagedata--object) objects to prepare the data for each stage to start the render process.|

Returns:
>|Name|Type|Description|
>|-|-|-|
>|Array(CancellablePromise)|Promise Array|An Array of CancellablePromise whose resolution will be a base64 image of the stage|

Example:
```
const renderStages = multiStageProcessor(renderer)(value)(stageData);
```

<br>
<br>

#### `stageProcessor(renderer:function;optional)(value:object)(stageData:object) : Cancellable Promise`
Receives:
>|Name|Type|Description|
>|-|-|-|
>|renderer|Function|A custom render function that will resolve into a base64 image of a stage.<br>Said function will receive as arguments an object containing the structure and ui.|
>|value|Object|A [value](#value--object) object containing the user changes|
>|stageData|Array|A [stageData](#stagedata--object) object to prepare the data for the stage to start the render process.|

Returns:
>|Name|Type|Description|
>|-|-|-|
>|CancellablePromise|Promise|A cancellablePromise whose resolution will be a base64 image of the stage|


Example:

    const renderStage = stageProcessor(renderer)(value)(stageData);

<br>
<br>

#### variationsProcessor(renderer:function;optional)(value:object)(stageData:array) : Array(Cancellable Promise)

Receives:
|Name|Type|Description|
|-|-|-|
|renderer|Function|A custom render function that will resolve into a base64 image of a stage.<br>Said function will receive as arguments an object containing the structure and ui.|
|value|Array|An array of [value](#value--object) objects containing the user changes|
|stageData|Object|A [stageData](#stagedata--object) object to prepare the data for the stage to start the render process.


Returns:
|Name|Type|Description|
|-|-|-|
|Array(CancellablePromise)|Promise Array|An Array of CancellablePromise whose resolution will be a base64 image of the stage|


Example:

    const renderVaritaions = variationsProcessor(renderer)(stageData)(values);

