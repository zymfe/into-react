React 的 github 仓库地址：[React](https://github.com/facebook/react)，不管怎么样，先 clone 到本地再说。

看源码，先找到这个项目的 package.json，看它的 scripts 字段，了解它的构建命令。我们可以从 build 命令开始：

``` javascript
{
  "build": "node ./scripts/rollup/build.js"
}
```

根据路径，找到 /scripts/rollup/build.js 文件，build 文件主要执行 buildEverything 函数，部分代码如下：

``` javascript
if (!argv['unsafe-partial']) {
  await asyncRimRaf('build');
}

let bundles = [];
// eslint-disable-next-line no-for-of-loops/no-for-of-loops
for (const bundle of Bundles.bundles) {
  bundles.push(
    [bundle, UMD_DEV],
    [bundle, UMD_PROD],
    [bundle, UMD_PROFILING],
    [bundle, NODE_DEV],
    [bundle, NODE_PROD],
    [bundle, NODE_PROFILING],
    [bundle, RN_OSS_DEV],
    [bundle, RN_OSS_PROD],
    [bundle, RN_OSS_PROFILING]
  );

  // ... 省略

  // eslint-disable-next-line no-for-of-loops/no-for-of-loops
  for (const [bundle, bundleType] of bundles) {
    await createBundle(bundle, bundleType);
  }
}
```  

从 import 可以看到，Bundles.bundles 定义在 /scripts/rollup/bundles.js 文件中，部分代码如下：

``` javascript
const bundles = [
  /******* Isomorphic *******/
  {
    bundleTypes: [
      UMD_DEV,
      UMD_PROD,
      UMD_PROFILING,
      NODE_DEV,
      NODE_PROD,
      FB_WWW_DEV,
      FB_WWW_PROD,
      FB_WWW_PROFILING,
    ],
    moduleType: ISOMORPHIC,
    entry: 'react',
    global: 'React',
    externals: [],
  },
  // ... 省略其他打包配置
];
```

bundles 是一个数组，每个 bundle 就是一种一个打包配置项。

再来看 UMD_DEV 的指向：

``` javascript
const bundleTypes = {
  UMD_DEV: 'UMD_DEV',
  // ... 省略其他 bundle type
};
```

所以最后变量 bundles 的内容是：

``` javascript
let bundles = [
  [
    {
      bundleTypes: [
        UMD_DEV,
        UMD_PROD,
        UMD_PROFILING,
        NODE_DEV,
        NODE_PROD,
        FB_WWW_DEV,
        FB_WWW_PROD,
        FB_WWW_PROFILING,
      ],
      moduleType: ISOMORPHIC,
      entry: 'react',
      global: 'React',
      externals: [],
    }, // bundle
    UMD_DEV // bundleType
  ],
  [/*其他配置*/],
  [/*其他配置*/],
  [/*其他配置*/]
]
```

接着执行：

``` javascript
for (const [bundle, bundleType] of bundles) {
  await createBundle(bundle, bundleType);
}
```

找到 createBundle 函数：

``` javascript
async function createBundle(bundle, bundleType) {
  if (shouldSkipBundle(bundle, bundleType)) {
    return;
  }
  // filename = react.development.js
  const filename = getFilename(bundle.entry, bundle.global, bundleType);
  const logKey =
    chalk.white.bold(filename) + chalk.dim(` (${bundleType.toLowerCase()})`);
  // format = umd
  const format = getFormat(bundleType);
  // packageName = react
  const packageName = Packaging.getPackageName(bundle.entry);

  // ... 省略

  const [mainOutputPath, ...otherOutputPaths] = Packaging.getBundleOutputPaths(
    bundleType, // UMD_DEV
    filename,   // react.development.js
    packageName // react
  );
}
```

Packaging.getBundleOutputPaths 最终返回的是：

``` javascript
return [
  `build/node_modules/${packageName}/umd/${filename}/`,
  `build/dist/${filename}`,
];
```

即上文：

``` javascript
mainOutputPath = 'build/node_modules/react/umd/react.development.js'
```

验证一下：

执行 npm install 会报错，可以先安装 yarn：sudo npm install yarn -g，然后执行 npm install、npm run build 即可。

### 注意

本文最后编辑于2020/01/02，技术更替飞快，文中部分内容可能已经过时，如有疑问，可在线提issue。
