[![](http://xeolabs.com/images/wiki/schependomlaan.png)](http://xeogl.org/examples/#importing_gltf_Schependomlaan)

Schependomlaan IFC model converted to glTF and viewed with xeogl - [[Run demo](http://xeogl.org/examples/#importing_gltf_Schependomlaan)] 

# Contents

- [V0.8 Overview](#v08-overview)
- [Featured Apps](#featured-apps)
  * [BIMData.io](#bimdataio)
  * [SolidComponents](#solidcomponents)
- [New Features](#new-features)
  * [Scene graphs](#scene-graphs)
    + [Object hierarchy](#object-hierarchy)
    + [Models within object hierarchies](#models-within-object-hierarchies)
    + [Semantic data models](#semantic-data-models)
  * [GLTFModel enhancements](#gltfmodel-enhancements)
    + [handleNode callback](#handlenode-callback)
    + [Generating IDs for Objects](#generating-ids-for-objects)
  * [Edge emphasis effect](#edge-emphasis-effect)
- [API changes](#api-changes)
  * [Transform hierarchies](#transform-hierarchies)
  * [Simplified boundary management](#simplified-boundary-management)
  * [Removed OBJGeometry and Nintendo3DSGeometry](#removed-objgeometry-and-nintendo3dsgeometry)
  * [Less scene mutability](#less-scene-mutability)
  * [Globally-defined custom clipping planes](#globally-defined-custom-clipping-planes)
  * [Globally-defined light sources](#globally-defined-light-sources)

# V0.8 Overview

This release simplifies xeogl and reduces its memory footprint, allowing us to load larger models, and to load them a bit more quickly.

**This release contains several breaking changes** but hopefully this document should help you to update your apps.

The main changes are:

* A conventional scene graph [Object](http://xeogl.org/docs/classes/Object.html) hierachy, in which each Object defines its local modeling transform - *(see [Scene Graphs](#scene-graphs))*.
* Renamed ````Entity```` to [Mesh](http://xeogl.org/docs/classes/Mesh.html), which also subclasses [Object](http://xeogl.org/docs/classes/Object.html) in order to plug into the scene graph.
* Translate, Rotate and Scale components have been removed in favor of the scene graph.
* [GLTFModels](http://xeogl.org/docs/classes/GLTFModel.html) are now [Objects](http://xeogl.org/docs/classes/Object.html) that plug into the scene graph, containing child [Objects](http://xeogl.org/docs/classes/Objects.html) of their own, that represent their glTF model's ````scene```` ````node```` elements. They can also be configured with a callback to determine how their child [Object](http://xeogl.org/docs/classes/Object.html) hierarchies are created as they load the ````node```` elements - *(see [GLTFModel Enhancements](#gltfmodel-enhancements))*.
* Components for light sources and user clipping planes now apply globally. Just instantiate them to apply them to the scene - *(see [Globally-defined custom clipping planes](#globally-defined-custom-clipping-planes) and [Globally-defined light sources](#globally-defined-light-sources))*.
* Simpler boundary tracking - reduced to the World-space AABB of each Scene, Model, Object and Geometry. Object-aligned bounding boxes (OBB) have been removed *(see [Simplified boundary management](#simplified-boundary-management)).*
 
### V0.7.2 archived

In case you still need [v0.7.2](https://github.com/xeolabs/xeogl/releases/tag/v0.7.2), I've tagged it and archived its API docs and examples online for reference: 

* Release: https://github.com/xeolabs/xeogl/releases/tag/v0.7.2
* Docs: http://xeolabs.com/xeogl-v0.7/docs
* Examples: http://xeolabs.com/xeogl-v0.7/examples

# Featured Apps

There are two commercial apps in production using xeogl v0.8.

## BIMData.io

[![](http://xeolabs.com/images/bimdata/bimdata.png)](http://www.bimdata.io/en/bim-data-en/)

The **BIMData.io** 3D BIM viewer is built on  xeogl v0.8. The viewer loads IFC models from glTF and has many cool gadgets to help navigate your BIM (some still in development). 

[[Visit BIMData.io >>](http://www.bimdata.io/en/bim-data-en/)]

## SolidComponents

[![](http://xeolabs.com/images/solidcomponents/solidcomponents-mac.png)](http://xeolabs.com/portfolio/solidcomponents-viewer)

The **SolidComponents** online 3D CAD viewer is also built on xeogl v0.8. This viewer provides SolidComponents customers with an interactive 3D preview of each product within their online product catalog. 

[[Read more >>](http://xeolabs.com/portfolio/solidcomponents-viewer)]

# New Features

## Scene graphs

V0.8 introduces scene graphs. The main component for these is [Object](http://xeogl.org/docs/classes/Object.html), which is subclassed by:

* [Mesh](http://xeogl.org/docs/classes/Mesh.html), which represents a drawable 3D primitive. In V0.7, this component was called "Entity".
* [Group](http://xeogl.org/docs/classes/Group.html), which is a composite Object that represents a group of child Objects.
* [Model](http://xeogl.org/docs/classes/Model.html), which is a Group and is subclassed by [GLTFModel](http://xeogl.org/docs/classes/GLTFModel.html), [STLModel](http://xeogl.org/docs/classes/STLModel.html), [OBJModel](http://xeogl.org/docs/classes/OBJModel.html) etc. A Model can contain child Groups and Meshes that represent its parts.

As shown in the examples below, these component types can be connected into flexible scene hierarchies that contain content loaded from multiple sources and file formats.

An Object defines its local modeling transform, which is relative to its parent Object. This was formerly done by Translate, Rotate, Scale components, which have been removed in xeogl v0.8.

### Object hierarchy 

[![object hierarchies](http://xeolabs.com/images/wiki/objectHierarchy.png)](http://xeogl.org/examples/#objects_hierarchy)

[[Run demo](http://xeogl.org/examples/#objects_hierarchy)]

```` javascript
var boxGeometry = new xeogl.BoxGeometry();

var table = new xeogl.Group({
    id: "table",
    rotation: [0, 50, 0], position: [0, 0, 0], scale: [1, 1, 1],
    children: [
        new xeogl.Mesh({ // Red table leg
            id: "redLeg",
            position: [-4, -6, -4], scale: [1, 3, 1], rotation: [0, 0, 0],
            geometry: boxGeometry,
            material: new xeogl.PhongMaterial({
                diffuse: [1, 0.3, 0.3]
            })
        }),

        new xeogl.Mesh({ // Green table leg
            id: "greenLeg",
            position: [4, -6, -4], scale: [1, 3, 1], rotation: [0, 0, 0],
            geometry: boxGeometry,
            material: new xeogl.PhongMaterial({
                diffuse: [0.3, 1.0, 0.3]
            })
        }),

        new xeogl.Mesh({// Blue table leg
            id: "blueLeg",
            position: [4, -6, 4], scale: [1, 3, 1], rotation: [0, 0, 0],
            geometry: boxGeometry,
            material: new xeogl.PhongMaterial({
                diffuse: [0.3, 0.3, 1.0]
            })
        }),

        new xeogl.Mesh({  // Yellow table leg
            id: "yellowLeg",
            position: [-4, -6, 4], scale: [1, 3, 1], rotation: [0, 0, 0],
            geometry: boxGeometry,
            material: new xeogl.PhongMaterial({
                diffuse: [1.0, 1.0, 0.0]
            })
        }),

        new xeogl.Mesh({ // Purple table top
            id: "tableTop",
            position: [0, -3, 0], scale: [6, 0.5, 6], rotation: [0, 0, 0],
            geometry: boxGeometry,
            material: new xeogl.PhongMaterial({
                diffuse: [1.0, 0.3, 1.0]
            })
        })
    ]
});

var scene = table.scene;

// Find Mesh by global ID on Scene
var redLeg = scene.components["redLeg"];

// Find mesh by global ID on parent Group
var greenLeg = table.childMap["greenLeg"];

// Find mesh by index in Group's child list
var blueLeg = table.children[2];

// Periodically update transforms
scene.on("tick", function () {

    // Rotate legs
    redLeg.rotateY(0.5);
    greenLeg.rotateY(0.5);
    blueLeg.rotateY(0.5);

    // Rotate table
    table.rotateY(0.5);
    table.rotateX(0.3);
});
````

We can find our Objects by ID in a dedicated map on their Scene:

````JavaScript
var redLeg = scene.objects["yellowLeg"];
````

### Models within object hierarchies

As mentioned above, [Models](http://xeogl.org/docs/classes/Model.html) are [Objects](http://xeogl.org/docs/classes/Object.html), which allows them to be included within scene graph hierarchies, to be shown, hidden, transformed, highlighted etc. as part of the hierarchy. The example below shows three different types of Model plugged into the same hierarchy, showing how to load content from multiple formats into your scene.

[![model hierarchies](http://xeolabs.com/images/wiki/modelHierarchy.png)](http://xeogl.org/examples/#objects_hierarchy_models)

[[Run demo](http://xeogl.org/examples/#objects_hierarchy_models)]

```` javascript
var modelsGroup = new xeogl.Group({
    rotation: [0,0,0], position: [0,0,0], scale: [1,1,1],

    children: [
        new xeogl.GLTFModel({
            id: "engine",
            src: "models/gltf/2CylinderEngine/glTF/2CylinderEngine.gltf",
            scale: [.2,.2,.2], position: [-110,0,0], rotation: [0,90,0]
        }),

        new xeogl.GLTFModel({
            id: "hoverBike",
            src: "models/gltf/hover_bike/scene.gltf",
            scale: [.5,.5,.5], position: [0,-40,0]
        }),

        new xeogl.STLModel({
            id: "f1Car",
            src: "models/stl/binary/F1Concept.stl",
            smoothNormals: true,
            scale: [3,3,3], position: [110,-20,60], rotation: [0,90,0]
         })
    ]
});

var scene = modelsGroup.scene;
var engine = scene.models["engine"]; // Find model by ID

scene.on("tick", function () {
    models.rotateY(.3); // Spin the whole group
    engine.rotateZ(.3); // Spin one of the models
});
````

### Semantic data models

[![object entities](http://xeolabs.com/images/wiki/entities.png)](http://xeogl.org/examples/#objects_entities)

[[Run demo](http://xeogl.org/examples/#objects_entities)]

In xeogl V0.8 we can organize our [Objects](http://xeogl.org/docs/classes/Object.html) using a generic conceptual data model that describes the semantics of our application domain. We do this by assigning "entity classes" to those Objects that we consider to be entities within our domain, and then we're able to reference those Objects according to their entity classes. Click the screenshot above to try a demo of what we can do with this mechanism.

```` Javascript
var boxGeometry = new xeogl.BoxGeometry();

var table = new xeogl.Group({
    id: "table",
    rotation: [0,50,0], position: [0,0,0], scale: [1,1,1],

    children: [
        new xeogl.Mesh({ // Red table leg
            id: "redLeg",
            entityType: "supporting", // <<------------ Entity class
            position: [-4,-6,-4], scale:[1,3,1], rotation: [0,0,0],
            geometry: boxGeometry,
            material: new xeogl.PhongMaterial({
                diffuse: [1, 0.3, 0.3]
            })
        }),

        new xeogl.Mesh({ // Green table leg
            id: "greenLeg",
            entityType: "supporting", // <<------------ Entity class
            position: [4,-6,-4], scale: [1,3,1], rotation: [0,0,0],
            geometry: boxGeometry,
            material: new xeogl.PhongMaterial({
                diffuse: [0.3, 1.0, 0.3]
            })
        }),

        new xeogl.Mesh({// Blue table leg
            id: "blueLeg",
            entityType: "supporting", // <<------------ Entity class
            position: [4,-6,4], scale: [1,3,1], rotation: [0,0,0],
            geometry: boxGeometry,
            material: new xeogl.PhongMaterial({
                diffuse: [0.3, 0.3, 1.0]
            })
        }),

        new xeogl.Mesh({  // Yellow table leg
            id: "yellowLeg",
            entityType: "supporting", // <<------------ Entity class
            position: [-4,-6,4], scale: [1,3,1], rotation: [0,0,0],
            geometry: boxGeometry,
            material: new xeogl.PhongMaterial({
                diffuse: [1.0, 1.0, 0.0]
            })
        })

        new xeogl.Mesh({ // Purple table top
            id: "tableTop",
            entityType: "surface", // <<------------ Entity class
            position: [0,-3,0], scale: [6,0.5,6], rotation: [0,0,0],
            geometry: boxGeometry,
            material: new xeogl.PhongMaterial({
                diffuse: [1.0, 0.3, 1.0]
            })
        })
    ]
});
````

We can find our entities in a dedicated map, that contains only the Objects that have the "entityType" property set:

````JavaScript
var yellowLegMesh = scene.entities["yellowLeg"];
````

We can get a map of all Objects of a given entity class:

````JavaScript
var supportingEntities = scene.entityTypes["supporting"];
var yellowLegMesh = supportingEntities["yellowLeg"];
````

We can do state updates on entity Objects by their entity class, in a batch:

````JavaScript
scene.setVisible(["supporting"], false);               // Hide the legs
scene.setVisible(["supporting"], true);                // Show the legs again
scene.setHighlighted(["supporting", "surface"], true); // Highlight legs & table top
````

The Scene also has convenience maps dedicated to tracking the visibility, ghosted, highlighted and selected states of entity Objects:

````JavaScript
var yellowLegMesh = scene.visibleEntities["yellowLeg"];
var isYellowLegVisible = yellowLegMesh !== undefined;
yellowLegMesh.highlighted = false;
var isYellowLegHighlighted = scene.highlightedEntities["yellowLeg"];
````

## GLTFModel enhancements

[GLTFModels](http://xeogl.org/docs/classes/GLTFModel.html) are now [Objects](http://xeogl.org/docs/classes/Object.html) that plug into the scene graph, containing child [Objects](http://xeogl.org/docs/classes/Objects.html) of their own, that represent their glTF model's ````scene```` ````node```` elements (see [Scene Graphs](#scene-graphs)). 

[GLTFModels](http://xeogl.org/docs/classes/GLTFModel.html) can also be configured with a ````handleNode```` callback to determine how their child [Object](http://xeogl.org/docs/classes/Object.html) hierarchies are created as they load the ````node```` elements. 

### handleNode callback

As a GLTFModel parses glTF, it creates child Objects from the ````node```` elements in the glTF ````scene````. 

GLTFModel traverses the ````node```` elements in depth-first order. We can configure a GLTFModel with a ````handleNode```` callback to call at each ````node````, to indicate how to process the ````node````.

Typically, we would use the callback to selectively create Objects from the glTF ````scene````, while maybe also configuring those Objects depending on what the callback finds on their ````node```` elements. 

For example, we might want to load a building model and set all its wall objects initially highlighted. For ````node```` elements that have some sort of attribute that indicate that they are walls, then the callback can indicate that the GLTFMOdel should create Objects that are initially highlighted.

The callback accepts two arguments:

* ````nodeInfo```` - the glTF node element.
* ````actions```` - an object on to which the callback may attach optional configs for each Object to create.

When the callback returns nothing or ````false````, then GLTFModel skips the given ````node```` and its children. 

When the callback returns ````true````, then the GLTFModel may process the ````node````. 

For each Object to create, the callback can specify initial properties for it by creating a ````createObject```` on its ````actions```` argument, containing values for those properties.

In the example below, we're loading a GLTF model of a building. We use the callback create Objects only for ````node```` elements who name is not "dontLoadMe". For those Objects, we set them highlighted if their ````node``` element's name happens to be "wall".
 
````Javascript
var houseModel = new xeogl.GLTFModel({
    src: "models/myBuilding.gltf",
   
    // Callback to intercept creation of objects while parsing glTF scene nodes

    handleNode: function (nodeInfo, actions) {

        var name = nodeInfo.name;

        // Don't parse glTF scene nodes that have no "name" attribute, 
        // but do continue down to parse their children.
        if (!name) {
            return true; // Continue descending this node subtree
        }

        // Don't parse glTF scene nodes named "dontLoadMe",
        // and skip their children as well.
        if (name === "dontLoadMe") {
            return false; // Stop descending this node subtree
        }    
       
        // Create an Object for each glTF scene node. 

        // Highlight the Object if the name is "wall"

        actions.createObject = {
            highlighted: name === "wall"
        };

        return true; // Continue descending this glTF node subtree
    }
});
````

### Generating IDs for Objects

You can use the ````handleNodeNode```` callback to generate a unique and deterministic ID for each Object:

````javascript
var model = new xeogl.GLTFModel({

    id: "gearbox",
    src: "models/gltf/gearbox_conical/scene.gltf",

    handleNode: (function() {
        var objectCount = 0;
        return function (nodeInfo, actions) {
            if (nodeInfo.mesh !== undefined) { // Node has a mesh
                actions.createObject = {
                    id: "gearbox." + objectCount++
                };
            }
            return true;
        };
    })()
});

// Highlight a couple of Objects by ID
model.on("loaded", function () {
    model.objects["gearbox.83"].highlighted = true;
    model.objects["gearbox.81"].highlighted = true;
});
````

[[Run demo](http://xeogl.org/examples/#effects_demo_gearbox)]

If the ````node```` elements have ````name```` attributes, then we can use those names with the ````handleNodeNode```` callback to generate a (hopefully) unique ID for each Object:

````javascript
var adamModel = new xeogl.GLTFModel({

    id: "adam",
    src: "models/gltf/adamHead/adamHead.gltf",

    handleNode: function (nodeInfo, actions) {
        if (nodeInfo.name && nodeInfo.mesh !== undefined) { // Node has a name and a mesh
            actions.createObject = {
                id: "adam." + nodeInfo.name
            };
        }
        return true;
    }
});

// Highlight a couple of Objects by ID
model.on("loaded", function () {
    model.objects["adam.node_mesh_Adam_mask_-4108.0"].highlighted = true;
    model.objects["adam.node_Object001_-4112.5"].highlighted = true;
});
````
[[Run demo](http://xeogl.org/examples/#effects_demo_adam)]

## Edge emphasis effect

[![](http://xeolabs.com/images/wiki/schependomlaan.png)](http://xeogl.org/examples/#importing_gltf_Schependomlaan)

[[Run demo](http://xeogl.org/examples/#importing_gltf_Schependomlaan)] 

The [Object](http://xeogl.org/docs/classes/Object.html) component has a flag that emphasizes its edges, which is is inherited by its [Mesh](http://xeogl.org/docs/classes/Mesh.html) and [Model](http://xeogl.org/docs/classes/Model.html) subclasses. In this example we'll show edges on the Schependomlaan IFC model, which has been converted to glTF so that we can load it into xeogl. 

````javascript
var model = new xeogl.GLTFModel({
    src: "models/schependomlaan.gltf"
});

model.on("loaded", function() {
    model.edges = true;
});
````

No edges | Medium edges |  Strong edges
------------ | ------------- | -------------
[![edges](http://xeolabs.com/images/wiki/edges_none.jpg)](http://xeogl.org/examples/#importing_gltf_OfficePlan) | [![edges](http://xeolabs.com/images//wiki/edges_medium.jpg)](http://xeogl.org/examples/#importing_gltf_OfficePlan) | [![edges](http://xeolabs.com/images//wiki/edges_strong.jpg)](http://xeogl.org/examples/#importing_gltf_OfficePlan)

# API changes

## Transform hierarchies 

Translate, Rotate and Scale components have been removed in xeogl v0.8 in favor of a more conventional scene graph [Object](http://xeogl.org/docs/classes/Object.html) tree, in which each Object defines its local modeling transform - see [Scene Graphs](#scene-graphs).


## Simplified boundary management

xeogl v0.8 provides only the World-space AABB of each [Scene](http://xeogl.org/docs/classes/Scene.html), [Model](http://xeogl.org/docs/classes/Model.html), [Object](http://xeogl.org/docs/classes/Object.html) and [Geometry](http://xeogl.org/docs/classes/Geometry.html). Object-aligned bounding boxes (OBB) have been removed.

````javascript
var aabb1 = myScene.aabb;         // Boundary of a xeogl.Scene
var aabb2 = myModel.aabb;         // Boundary of a xeogl.Model
var aabb3 = myObject.aabb;        // Boundary of a xeogl.Object
var aabb4 = myMesh.geometry.aabb; // Boundary of a xeogl.Geometry
````

We can subscribe to boundary updates as shown below. Note that the updated boundaries are not passed to our callbacks. Instead, our callbacks need to get them from the target components. This allows xeogl to defer calculation of the boundaries, to lazy-calculate them at the point that we actually get them. 

Boundaries could be continuously updating, so this saves xeogl from wastefully re-calculating boundaries that we never use, or use only periodically (eg. every Nth frame). 

````javascript
myScene.on("boundary", function() {
    var aabb = myScene.aabb; // <<--- Boundary is lazy-updated at this point
});

myModel.on("boundary", function() {
    var aabb = myModel.aabb;
});

myObject.on("boundary", function() {
    var aabb = myObject.aabb;
});

myGeometry.on("boundary", function() {
    var aabb = myGeometry.aabb;
});
````

## Removed OBJGeometry and Nintendo3DSGeometry

The OBJGeometry and Nintendo3DSGeometry components were Geometry subclasses that loaded themselves from OBJ and 3DS models. These are removed in v0.8 and replaced with loader functions, which are used as shown below.

````javascript
xeogl.loadOBJGeometry(http://xeogl.scene, "raptor.obj", function (geometry) {

    var mesh = new xeogl.Mesh({
        geometry: geometry,
        material: new xeogl.PhongMaterial({
            pointSize: 5,
            diffuseMap: new xeogl.Texture({
                src: "models/obj/raptor/raptor.jpg"
            })
        }),
        rotation: [0, 120, 0],
        position: [10, 3, 10]
    });
});
````

[[Run demo](http://xeogl.org/examples/#importing_obj_raptor)]

````javascript
xeogl.load3DSGeometry(http://xeogl.scene, "lexus.3ds", function (geometry) {

    var mesh = new xeogl.Mesh({
         geometry: geometry,
         material: new xeogl.PhongMaterial({
            emissive: [1, 1, 1],
            emissiveMap: new xeogl.Texture({ 
                src: "models/3ds/lexus.jpg"
            })
        }),
        rotation: [-90, 0, 0]
    });
});
````
[[Run demo](http://xeogl.org/examples/#importing_3ds_lexus)]

## Less scene mutability

Components like Materials and Meshes (formerly called Entity) cannot be plugged together dynamically; now they can only be composed once at instantiation, and you can no longer add or remove components to them dynamically.

No longer supported:

````JavaScript
var mesh = new xeogl.Mesh({
    geometry: new xeogl.TorusGeometry({
        radius: 1.0,
        tube: 0.3,
        radialSegments: 120,
        tubeSegments: 60
    })
});

mesh.material = new xeogl.MetallicMaterial();

mesh.material.baseColorMap: new xeogl.Texture({
    src: "textures/diffuse/uvGrid2.jpg"
})
````

Do this instead:

````JavaScript
var mesh = new xeogl.Mesh({
    geometry: new xeogl.TorusGeometry({
        radius: 1.0,
        tube: 0.3,
        radialSegments: 120,
        tubeSegments: 60
    }),
    material: new xeogl.MetallicMaterial({
        baseColorMap: new xeogl.Texture({
            src: "textures/diffuse/uvGrid2.jpg"
        })
    })
});
````

Components were originally editable like this because I wanted xeogl to form the basis for a graphical node-based scene editor. However, that use case never really materialized, and the mutability was complicating the code. Removing this editability reduces the scene's memory footprint and allows components to be constructed much faster. 

## Globally-defined custom clipping planes

[![clipping](http://xeolabs.com/images/wiki/clipping.png)](http://xeogl.org/examples/#effects_clipping)

[[Run demo](http://xeogl.org/examples/#effects_clipping)]

Clipping planes are now created by simply instantiating [Clip](http://xeogl.org/docs/classes/Clip.html) components, without attaching them to anything. The Clip components will then apply globally to all Meshes in the Scene.

````JavaScript
xeogl.scene.clearClips();

new xeogl.Clip({
    id: "clip0",
    pos: [0.8, 0.8, 0.8],
    dir: [-1, -1, -1],
    active: true
});

new xeogl.Clip({
    id: "clip1",
    pos: [0.8, 0.8, -0.8],
    dir: [-1, -1, 1],
    active: true
});

new xeogl.Clip({
    id: "clip2",
    pos: [0.8, -0.8, -0.8],
    dir: [-1, 1, 1],
    active: true
});
````

## Globally-defined light sources

[![lights](http://xeolabs.com/images/wiki/lights.png)](http://xeogl.org/examples/#lights_point_view_threePoint)

[[Run demo](http://xeogl.org/examples/#lights_point_view_threePoint)]

Light sources are also now created by simply instantiating their components, without attaching them to anything. The light sources will then apply globally to all Meshes in the Scene.

````Javascript
xeogl.scene.clearLights();

new xeogl.PointLight({
    id: "keyLight",
    pos: [-80, 60, 80],
    color: [1.0, 0.3, 0.3],
    intensity: 1.0,
    space: "view"
});

new xeogl.PointLight({
    id: "fillLight",
    pos: [80, 40, 40],
    color: [0.3, 1.0, 0.3],
    intensity: 1.0,
    space: "view"
});

new xeogl.PointLight({
    id: "rimLight",
    pos: [-20, 80, -80],
    color: [0.6, 0.6, 0.6],
    intensity: 1.0,
    space: "view"
});
````

Same thing for [ReflectionMap](http://xeogl.org/docs/classes/ReflectionMap.html) components:

````JavaScript
new xeogl.ReflectionMap({
    src: [
        "textures/Uffizi_Gallery_Radiance_PX.png",
        "textures/Uffizi_Gallery_Radiance_NX.png",
        "textures/Uffizi_Gallery_Radiance_PY.png",
        "textures/Uffizi_Gallery_Radiance_NY.png",
        "textures/Uffizi_Gallery_Radiance_PZ.png",
        "textures/Uffizi_Gallery_Radiance_NZ.png"
    ],
    encoding: "sRGB"
});
````

And ditto for [LightMap](http://xeogl.org/docs/classes/LightMap.html) components:

```` JavaScript
new xeogl.LightMap({
    src: [
        "textures/Uffizi_Gallery_Irradiance_PX.png",
        "textures/Uffizi_Gallery_Irradiance_NX.png",
        "textures/Uffizi_Gallery_Irradiance_PY.png",
        "textures/Uffizi_Gallery_Irradiance_NY.png",
        "textures/Uffizi_Gallery_Irradiance_PZ.png",
        "textures/Uffizi_Gallery_Irradiance_NZ.png"
    ],
    encoding: "linear"
});
````