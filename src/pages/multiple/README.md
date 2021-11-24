#### SkeletonUtils
- 骨架，蒙皮网格和骨骼操作的工具函数
```
//模型复制,如果需要大量复制，刻在下面进行循环
const clonedScene = SkeletonUtils.clone( model.scene );
// 根据mesh名字获取Mesh
const clonedMesh = clonedScene.getObjectByName( u.meshName );
```