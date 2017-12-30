# Unique Shadows

Unique Shadows is a rendering component developed for The Blacksmith to improve shadow resolution and filtering on the main characters. 


### How does it work?

First things first:
- Unique Shadows are only active in playmode. Once activated they work both in scene and game view. Keep reading below for an explanation as to why we enforce this annoying restriction.
- The component has been tested for D3D11, D3D9 and OGL2.

The component works on a "group" basis, and affects all child renderers of the owning gameobject. It works by creating unique material instances on Awake, and manages these to configure and toggle unique shadows. The primary reason we're creating unique material instances over, e.g. using material property blocks, is that we need to set shader keywords on a per-group basis - something that's not yet available through property blocks. This is also the reason we chose to activate them only in playmode; we didn't want to destructively modify the scene in edit mode.


### Setting up your own

The provided example scene should help you get started, but here's a quick setup guide for a new group:
- Start by adding the UniqueShadowSun component to your shadow casting light source. We chose to make this an explicit step so that we could pick which lights would active unique shadows, and which would just fall back to using the regular cascades. You can have as many of these as you like, but only one of them should ever be active at any one time.
- Add a UniqueShadow component to the root of the object hierarchy you'd like to cast unique shadow onto. 
- Add at least one shadow focus element and populate the target and radius fields. You might also wanna add a tiny bit of depth bias here. The reason we have have this concept of multiple explicit focus setups, is that we experimented with unique settings for each shot in The Blacksmith.
- Make sure the materials used in your target objects actually have a shader that supports unique shadows. There's one included in the project, and you can easily make your own.
- Press play and things should be working.


### Shader support, you say?

Yes, the materials used for unique shadows do need to a use a shader that has been prepared with support specifically for this feature.

Although the specifics might depend on how esoteric your shaders are, most shaders can be unique shadows enabled simply by adding these two lines:

	#pragma multi_compile _ UNIQUE_SHADOW UNIQUE_SHADOW_LIGHT_COOKIE
	#include "Path/To/UniqueShadow/UniqueShadow_ShadowSample.cginc"

This works both for surface and vertex/fragment shaders. Note, however, that the include needs to go above the include of any other builtin Unity include files.


### All the other options

The component contains quite a few configuration fields, here's a quick description of what they mean.

**Shadow Map Size**: Should be fairly self-explanatory; pick a size for your unique shadow map. There's a public function 'SetDownscale' on the component that allows you to programmatically change scale this size down for different quality settings.

**Culling Distance**: How far away from the rendering camera unique shadows should still be active. The general idea is that as objects move further away from the camera, they become smaller on screen, and at some distance it's just a waste of performance keeping unique shadows enabled for them.

**Inclusion Mask**: Which layers to render into unique shadow map. This is ignored when capturing scene, which instead relies on the culling mask of the currently active UniqueShadowSun light source.

**Use Scene Capture**: Whether to capture the world outside the focus radius and project it onto the shadow camera near plane. This is required to have the static world cast shadows onto dynamic, uniquely shadowed objects.

**Blocker Search Distance / Blocker Distance Scale / Light Near Size / Light Far Size**: These parameters control the penumbra/softness shadow filtering, and are only used when running D3D11.

**Fallback Filter Width**: This parameter controls the softness of the shadow filtering, and is used when NOT running D3D11.

**Start Focus**: Which focus settings to select by default. Normally just left at 0.

**Focus/Auto Focus**: Whether to calculate focus offset and radius based on all renderer bounds. (never used in The Blacksmith, YMMV)  

**Focus/Auto Focus Radiu Bias**: Add or subtract to the calculated focus radius.

**Focus/Target**: The transform to focus shadows around. Should be something that follows the group around, like the root for rigid objects, or the ref or hips for a character.

**Focus/Offset**: Focus translation offset relative to the target transform.

**Focus/Radius**: Focus radius. Should encompass all shadow receivers in the group, but not external shadow casters.

**Focus/Depth Bias**: Depth bias to avoid shadow acne. Should be a tiny positive number. 

**Focus/Scene Capture Distance**: How far towards the light source shadow casters should be captured.
