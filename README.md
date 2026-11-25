# Fast-Weight-Surface-Memory-for-3D-Fragment-Interpolation
Fast-Weight Surface Memory for 3D Fragment Interpolation

Research prototype / early concept

This project explores whether a small fast-weight surface memory can provide useful local surface context for fragment interpolation during 3D rasterization.

The idea is still preliminary. The current goal is to test whether a fixed-size fast-weight representation can efficiently provide approximate neighbourhood information to a large number of fragment queries.

Motivation

Traditional rasterization usually interpolates fragment attributes using information from the three vertices of the current triangle.

This is extremely efficient, but the available information is strongly local to the triangle. A fragment does not directly know how the surrounding surface behaves outside that triangle.

One possible solution is to explicitly collect neighbouring vertices or triangles for every fragment. However, many fragments may repeatedly query the same surface region, causing the same neighbourhood information to be gathered and processed many times.

The main idea of this project is therefore simple:

Collect local surface information once, compress it into a small fixed-size fast-weight memory, and allow many fragments to query the same memory in parallel.

The memory does not need to reconstruct every neighbouring vertex exactly. Instead, it is intended to provide a fast and approximate description of the surrounding surface.

Core Idea

A local surface region contains a variable number of samples. These samples may contain information such as position, normal, and other geometric attributes.

Each sample produces a spatial key and a value containing useful surface information.

The information is accumulated into a fixed-size fast-weight matrix.

S = sum of k_i v_i^T

The size of S does not directly depend on the number of samples in the surface region.

A fragment at position q then generates its own spatial key and queries the memory.

c(q) = k(q)^T S

The result c(q) represents approximate local surface context around the queried fragment.

Different fragment positions can retrieve different information from the same memory because the read operation depends on the fragment coordinate.

The fast-weight matrix therefore acts as a compact coordinate-addressed surface memory.

Why Fast-Weight Memory

The main purpose of the fast-weight memory is to obtain useful neighbourhood information without repeatedly gathering the complete neighbourhood for every fragment.

It has several useful properties.

The state has a fixed size.

The same state can be shared by many fragments.

Different fragment coordinates can retrieve different context from the same state.

The state can be constructed once and queried many times.

Fragment queries are independent and can be evaluated in parallel.

The fast-weight formulation can also be interpreted as a linear-attention-style operation. In this project, however, the fast-weight memory interpretation is more appropriate because the main goal is to construct a reusable spatial context state rather than to perform sequence attention.

Initial Prototype

The first prototype will focus on fragment normal correction.

Traditional barycentric interpolation first produces a baseline normal for each fragment.

The fast-weight surface memory provides additional information about the surrounding surface.

A small neural network then uses the baseline normal together with the retrieved surface context to predict a correction to the normal.

Conceptually, the process is:

Traditional fragment normal

plus

Fast-weight surface context

produces

Normal correction

produces

Corrected fragment normal

The corrected normal can then be used directly by an ordinary Phong, Blinn-Phong, or PBR shader.

The neural network does not need to predict the final rendered colour.

This keeps the initial experiment focused on whether fast-weight surface context can improve fragment attributes themselves.

Expected Information in the Memory

The memory is not intended to store an exact copy of the neighbourhood.

Instead, it may preserve approximate information such as the overall local normal direction, local surface orientation, whether the surrounding surface is relatively flat or curved, local curvature tendencies, and proximity to sharp geometric changes.

This approximate context may provide information that cannot be obtained from the current triangle alone.

Expected Visual Effect

If the predicted fragment normals are more consistent with the surrounding surface, the method may improve shading on coarse meshes without increasing the number of triangles.

Possible effects include smoother lighting transitions, more stable specular highlights, reduced visible faceting on curved surfaces, and fragment normals that more closely resemble those of a higher-resolution surface.

The geometry itself does not need to change.

The initial goal is therefore not to replace the renderer, but to improve the surface attributes supplied to a conventional renderer.

Parallelism

The method is intended to be highly parallel.

Surface samples can independently contribute information to the fast-weight state and their contributions can be combined.

After the state has been constructed, many fragments can query the same state independently.

For a group of fragment queries, the reads can be expressed as a matrix operation.

C = K_Q S

This structure is suitable for GPU parallelism.

The important idea is write once and read many.

A surface region may contain relatively few vertices while producing thousands of fragments on screen. In this situation, compressing the neighbourhood once and sharing the resulting memory may avoid repeatedly gathering the same neighbourhood information.

Large Meshes

Using one fast-weight matrix for an entire large mesh may cause unrelated surface regions to interfere with each other.

The current expectation is therefore to divide a mesh into local regions and construct a small fast-weight state for each region.

All regions can share the same learned encoder and decoder parameters.

Only the local fast-weight states are different.

The exact patching strategy is not fixed yet and will be investigated experimentally.

Possible Second-Order Information

A later version may include second-order geometric information.

First-order information may mainly describe local position and normal trends.

Second-order terms may provide additional information about local geometric variation and curvature.

This may help the memory represent how normals change across the surrounding surface rather than only their average direction.

Second-order information is currently considered an optional extension rather than a fixed part of the method.

Possible Extensions

If the initial normal-correction experiment is successful, the same surface memory may later be tested for other fragment attributes.

Possible examples include curvature, roughness, material parameters, ambient occlusion cues, and multi-scale surface context.

These are currently possible research directions rather than established parts of the method.

Current Research Questions

The initial prototype is intended to answer several basic questions.

Can neighbourhood context improve fragment normals beyond standard triangle-local interpolation?

Can a fixed-size fast-weight matrix retain enough useful information about a variable-size surface neighbourhood?

Does coordinate-dependent fast-weight memory provide more useful information than a simple pooled feature vector?

Can fast-weight memory approach the quality of explicit neighbourhood processing while avoiding repeated neighbourhood gathering for every fragment?

How does memory size affect reconstruction quality, runtime, and interference?

Can higher-order geometric information improve the representation of local surface shape?

Status

This repository represents an early research concept rather than a finalized method.

Many implementation details are intentionally left open until the basic hypothesis has been tested.

The central idea is:

Use a compact fast-weight surface memory to provide fast, approximate, fixed-size neighbourhood context for massively parallel 3D fragment interpolation.
