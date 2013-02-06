# voxel-engine

## A voxel engine in javascript using three.js

Learn more at http://voxeljs.com

Write a voxel.js game in browser: http://voxel-creator.jit.su

hello world code: http://github.com/maxogden/voxel-hello-world

# example

``` js
var createGame = require('voxel-engine')
var game = createGame()
var container = document.body
game.appendTo(container)
container.setupPointerLock(container)
```

# API

## require('voxel-engine')(options)

Returns a new game instance. `options` defaults to:

```javascript
{
  texturePath: './textures/',
  generate: function(x,y,z) {
    return x*x+y*y+z*z <= 20*20 ? 1 : 0 // sphere world
  },
  materials: [['grass', 'dirt', 'grass_dirt'], 'brick', 'dirt'],
  cubeSize: 25,
  chunkSize: 32,
  chunkDistance: 2,
  startingPosition: [35, 1024, 35],
  worldOrigin: [0,0,0],
  controlOptions: {jump: 6},
  controlLayout: 'qwerty',
  lightsDisabled: false,
  fogDisabled: false,
  generateChunks: true,
  mesher: voxel.meshers.greedy,
  playerHeight: 1.62
}
```

## Generating voxel worlds

Worlds have many chunks and chunks have many voxels. Chunks are cube shaped and are `chunkSize` x/y/z (default 32/32/32 - 32768 voxels per chunk). When the game starts it takes the `worldOrigin` and generates `chunkDistance` chunks in every x/y/z dimension (`chunkDistance` default of 2 means the game will render 2 chunks behind you, 2 in front etc for a total of 16 chunks.). 

There are three coordinate systems in voxel.js: 

- game coordinates (aka screen coordinates): every object added to a three.js scene gets a x/y/z position in game coordinates
- voxel coordinates: when generating the world or interacting with individual voxels you may need to refer to voxels in voxel coordinates. an example voxel coordinate might be [34, -50, 302] which would mean starting from the `worldOrigin` 34 voxels over, 50 down and 302 forward
- chunk coordinates: logically the same as voxel coordinates but for chunks. you probably won't need to use these as they are just used internally for rendering the world but it is good to know they exist.

When you create a game you can also pass functions that the game will ask for voxel data. Here is an example `generate` function that makes a randomly textured cube world with a diameter of 20 voxels:

```javascript
function generator(x, y, z) {
  if (x*x + y*y + z*z > 20*20) return 0
  return Math.floor(Math.random() * 4) + 1
}
```

The `generate` function will be called once for each voxel in the world. `x`, `y` and `z` will be values in voxel coordinates.

## Game events

There are a number of events you can listen to once you've instantiated a game. we use the node.js event emitter library which uses the following syntax for subscribing:

`emitter.on('eventName', function(arg1, arg2, etc) {})`

### game.on('tick', function(delta) {})

emits every time the game renders (usually no more than 60 times a second). delta is the time in milliseconds between this render and the last render

### game.voxelRegion.on('change', function(pos) {})

emits when you move between voxels. pos has x, y, and z voxel coordinates of the voxel you just entered

### game.chunkRegion.on('change', function(pos) {})

emits when you move between chunks. pos has x, y, and z chunk coordinates of the chunk you just entered

### game.on('missingChunk', function(chunkPosition) {})

emits when the player moves into range of a chunk that isn't loaded yet. if your game has `generateChunks` set to true it will automatically create the chunk and render it but if you are providing your own chunk generation then you can use this to hook into the game.

## Interacting with the voxel world

When the game renders it draws each voxel at `cubeSize` wide in three.js world coordinates (something like pixels wide). So a default chunk is 32 (`chunkSize`) * 25 (default `cubeSize`) === 800 wide.

To get the players current position you can do `gameInstance.controls.yawObject.position`. This returns a THREE.js Vector3 object (which just means an object with 'x', 'y', and 'z'). The coordinates are in world coordinates.

To look up the chunk at some world coordinates:

`gameInstance.voxels.chunkAtPosition(position)`

To look up the voxel at some world coordinates (relative to that voxels chunk):

`gameInstance.voxels.voxelVector(position)`

Create a brand new voxel at some world coordinates. Intended for use in first player contexts as it checks if a player is standing in the way of the new voxel. If you don't care about that then just use `setBlock`:

`gameInstance.createBlock(pos, val)`

`val` can be 0 or you can also use any single digit integer 0-9. These correspond to the materials array that you pass in to the game.

Set the value of a voxel at some world coordinates:

`gameInstance.setBlock(pos, val)`

Get the value of a voxel at some world coordinates:

`gameInstance.getBlock(pos)`

If you wanna see the lower level API for voxel data manipulation look at `chunker.js` inside the voxel module.

## license

BSD
