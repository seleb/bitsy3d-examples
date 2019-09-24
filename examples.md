# 3D Hack Examples

All examples provided use the same gamedata, have the 3d hack applied, and only change the `hackOptions` to produce different results.
The example gamedata is a modified version of [lunar cultivation](https://seansleblanc.itch.io/lunar-cultivation) which includes two rooms with a handful of varied tiles, sprites, and items.

## standard 3rd person

[demo](https://seans.site/stuff/bitsy-3d-hack-examples/standard%203rd%20person.html)

![standard 3rd person](https://i.imgur.com/kLz7A8V.png)

camera follows player and can be freely rotated and zoomed

```js
init: function(scene) {
	scene.activeCamera = makeBaseCamera();
	makeFollowPlayer(scene.activeCamera);
	addControls(scene.activeCamera);
},
```

## locked-angle 3rd person

[demo](https://seans.site/stuff/bitsy-3d-hack-examples/locked-angle%203rd%20person.html)

![locked-angle 3rd person](https://i.imgur.com/Ar8rmX7.png)

camera follows player, but can't be rotated or zoomed

```js
init: function (scene) {
	scene.activeCamera = makeBaseCamera(); // creates a camera with some basic presets
	makeFollowPlayer(scene.activeCamera); // locks the camera to the player
	// set the angle/zoom
	// alpha is yaw, beta is pitch, radius is zoom
	scene.activeCamera.alpha = Math.PI * 0.5;
	scene.activeCamera.beta = Math.PI * 0.4;
	scene.activeCamera.radius = 10;
},
```

## fixed 3rd person

[demo](https://seans.site/stuff/bitsy-3d-hack-examples/fixed%203rd%20person.html)

![fixed 3rd person](https://i.imgur.com/gFerZjg.png)

camera is placed at a fixed point and rotates to look at the player (note how this example creates a custom camera instead of using `makeBaseCamera`)

```js
init: function (scene) {
	var camera = new BABYLON$1.UniversalCamera("Camera", BABYLON$1.Vector3.Zero(), exports.scene);
	// perspective clipping
	camera.minZ = 0.001;
	camera.maxZ = bitsy.mapsize * 2;
	camera.position = new BABYLON$1.Vector3(8,10,-8);
	makeFollowPlayer(camera);
	scene.activeCamera = camera;
},
```

## first-person dungeon crawler

[demo](https://seans.site/stuff/bitsy-3d-hack-examples/first-person%20dungeon%20crawler.html)

![first-person dungeon crawler](https://i.imgur.com/Tc2lify.png)

camera is placed directly on the player, and is rotated with left/right movement (note that you'll typically want a blank avatar for this setup, or they may be visible when turning)

```js
cameraRelativeMovement: true,
tankControls: true,
init: function(scene) {
	scene.activeCamera = makeBaseCamera();
	makeFollowPlayer(scene.activeCamera);
	scene.activeCamera.lowerRadiusLimit = scene.activeCamera.upperRadiusLimit = 0.0001;
	scene.activeCamera.beta = Math.PI/2;
},
```

## isometric

[demo](https://seans.site/stuff/bitsy-3d-hack-examples/isometric.html)

![isometric](https://i.imgur.com/NaVlsuu.png)

camera uses a fixed orthographic perspective

```js
cameraRelativeMovement: false,
init: function (scene) {
	var camera = new BABYLON$1.UniversalCamera("Camera", BABYLON$1.Vector3.Zero(), scene);
	makeOrthographic(camera, bitsy.mapsize * Math.sqrt(2));
	// place on bottom left corner
	camera.position.x = -0.5;
	camera.position.y = bitsy.mapsize/2;
	camera.position.z = 0.5;
	// point towards center at typical isometric angle
	camera.rotation.x = Math.atan(0.5);
	camera.rotation.y = Math.PI / 4;
	scene.activeCamera = camera;
},
```

## fog

[demo](https://seans.site/stuff/bitsy-3d-hack-examples/fog.html)

![fog](https://i.imgur.com/VL6uvzh.png)

demonstrates the `addFog` helper

```js
init: function(scene) {
	scene.activeCamera = makeBaseCamera();
	makeFollowPlayer(scene.activeCamera);
	addControls(scene.activeCamera);
	// add fog starting 10% of the map's size away from the camera, and ending at 50% of the map's size
	addFog(0.1, 0.5);
},
```

## post-processing

[demo](https://seans.site/stuff/bitsy-3d-hack-examples/post-processing.html)

![post-processing](https://i.imgur.com/T4BbDw1.png)

demonstrates the `addShader` helper (example provided is a 1-bit 4x4 bayer dither shader)

```js
init: function(scene) {
	scene.activeCamera = makeBaseCamera();
	makeFollowPlayer(scene.activeCamera);
	addControls(scene.activeCamera);
	addShader(`
#ifdef GL_ES
	precision highp float;
#endif

// Samplers
varying vec2 vUV;
uniform sampler2D textureSampler;

// Parameters
uniform vec2 screenSize;

// Constants
const float posterize = 1.0;
const mat4 ditherTable = mat4(
	1, 8, 2, 10,
	12, 4, 14, 6,
	3, 11, 1, 9,
	16, 7, 13, 5
)/16.0;

void main(void) {
	vec3 col = texture2D(textureSampler, vUV).rgb;
	vec3 raw = vec3(col.r+col.g+col.b)/3.0;
	vec3 posterized = raw - mod(raw, 1.0/posterize);

	vec2 dit = floor(mod(vUV * screenSize, 4.0));
	float limit = ditherTable[int(dit.y)][int(dit.x)];
	vec3 dither = step(limit, (raw - posterized) * posterize) / posterize;

	col.rgb = vec3(posterized + dither);

	gl_FragColor = vec4(col.rgb, 1.0);
}`, 1.0);
},
```

## alternate 3D objects

[demo](https://seans.site/stuff/bitsy-3d-hack-examples/alternate%203D%20objects.html)

![alternate 3D objects](https://i.imgur.com/nNMZZ7c.png)

customizes `getType` and `getBillboardMode` to give a different look and feel with customizations specific to the example gamedata (e.g. water tiles are walls, which would ordinarily use `'box'`, but are overriden to use type `'floor'` instead)

```js
getType: function (drawing) {
	var drw = drawing.drw;
	var name = drawing.name || '';
	if (drawing.id === bitsy.playerId) {
		return 'plane';
	}
	if (name === 'water') {
		return 'floor';
	}
	if (name === 'seat') {
		return 'plane';
	}
	if (drw.startsWith('ITM') || drw.startsWith('SPR')) {
		return 'billboard';
	}
	if (['black', 'tree'].includes(name)) {
		return 'tower3';
	}
	if (drawing.isWall) {
		return 'box';
	}
	return 'floor';
},
getBillboardMode: function (BABYLON) {
	return BABYLON.TransformNode.BILLBOARDMODE_ALL;
},
```
