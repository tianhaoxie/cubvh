# cuBVH

A CUDA Mesh BVH acceleration toolkit.

### Install

```python
pip install git+https://github.com/ashawkey/cubvh

# or locally
git clone https://github.com/ashawkey/cubvh
cd cubvh
pip install .
```

### Usage

```python
import numpy as np
import trimesh
import torch

import cubvh

### build BVH from mesh
mesh = trimesh.load('example.ply')
# NOTE: you need to normalize the mesh first, since the max distance is hard-coded to 10.
BVH = cubvh.cuBVH(mesh.vertices, mesh.faces) # build with numpy.ndarray/torch.Tensor

### query ray-mesh intersection
rays_o, rays_d = get_ray(pose, intrinsics, H, W) # [N, 3], [N, 3], query with torch.Tensor (cuda)
intersections, face_id, depth = BVH.ray_trace(rays_o, rays_d) # [N, 3], [N,], [N,]

### query unsigned distance
points # [N, 3]
# uvw is the barycentric corrdinates of the closest point on the closest face (None if `return_uvw` is False).
distances, face_id, uvw = BVH.unsigned_distance(points, return_uvw=True) # [N], [N], [N, 3]

### query signed distance (INNER is NEGATIVE!)
# for watertight meshes (default)
distances, face_id, uvw = BVH.signed_distance(points, return_uvw=True, mode='watertight') # [N], [N], [N, 3]
# for non-watertight meshes:
distances, face_id, uvw = BVH.signed_distance(points, return_uvw=True, mode='raystab') # [N], [N], [N, 3]
```


Example for a mesh normal renderer by `ray_trace`:

```bash
python test/renderer.py # default, show a dodecahedron
python test/renderer.py --mesh example.ply # show any mesh file
```

https://user-images.githubusercontent.com/25863658/183238748-7ac82808-6cd3-4bb6-867a-9c22f8e3f7dd.mp4


### Acknowledgement

* Credits to [Thomas Müller](https://tom94.net/)'s amazing [tiny-cuda-nn](https://github.com/NVlabs/tiny-cuda-nn) and [instant-ngp](https://github.com/NVlabs/instant-ngp)!
