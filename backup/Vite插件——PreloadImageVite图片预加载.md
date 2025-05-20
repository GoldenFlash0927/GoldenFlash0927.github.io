# preloadImageVite图片预加载

---

### 1. **插件功能**

+ **预加载图片**：在构建时扫描指定目录下的图片，并在 HTML 中注入 `<link rel="preload">` 标签，提升图片加载速度。
+ **可配置项**：
  - `rel`：link 标签的 rel 属性（默认 `'preload'`）。
  - `imageDir`：图片目录路径（默认 `'../src/assets/image'`）。

---

### 2. **核心实现**

+ **钩子**：使用 Vite 的 `transformIndexHtml` 钩子，在构建时修改 HTML 文件。
+ **扫描图片**：通过 `fs.readdirSync` 读取图片目录，过滤出图片文件（支持 jpg、jpeg、png、gif、svg）。
+ **生成标签**：为每个图片生成 `<link rel="preload" href="..." as="image" />` 标签。
+ **注入 HTML**：将生成的标签注入到 HTML 的 `</head>` 标签前。

```typescript
import type { Plugin } from 'vite';
import { fileURLToPath } from 'url';
import { dirname, resolve } from 'path';
// 读取目录下所有文件
import { readdirSync } from 'fs';

// 获取当前文件的绝对路径
const __filename = fileURLToPath(import.meta.url);
// 获取当前文件的目录
const __dirname = dirname(__filename);

/**
 * Vite 图片预加载插件
 * @param options 配置项
 *   - rel: link 标签的 rel 属性，默认 'preload'(预加载) 或 'prefetch'(预获取)
 *   - imageDir: 图片目录（绝对路径或相对本插件文件的路径），默认 '../src/assets/image'
 */
interface PreloadImageViteOptions {
  rel?: string;
  imageDir?: string;
}

export default function preloadImageVite(options: PreloadImageViteOptions = {}): Plugin {
  const {
    rel = 'preload',
    imageDir = '../src/assets/image',
  } = options;

  return {
    name: 'vite-plugin-preload-image',
    // enforce: 'post', // 控制插件执行时机
    // apply: 'build', // 控制插件在哪个阶段生效
    /**
     * transformIndexHtml 钩子：在构建时修改 HTML 文件
     * @param html 原始 HTML 字符串
     * @returns 修改后的 HTML 字符串，注入 preload 标签
     */
    transformIndexHtml(html) {
      // 解析图片目录的绝对路径
      const absImageDir = resolve(__dirname, imageDir);
      // 读取目录下所有图片文件
      const imageFiles = readdirSync(absImageDir).filter(file => /\.(jpg|jpeg|png|gif|svg|webp)$/i.test(file));
      // 生成 preload link 标签
      const preloadTags = imageFiles.map(file => {
        const href = `${imageDir}/${file}`;
        return `<link rel="${rel}" href="${href}" as="image" />`;
      }).join('\n');
      // 注入到 </head> 前
      return html.replace('</head>', `${preloadTags}\n</head>`);
    },
  };
}

```

---

### 3. **配置项说明**

+ `rel`：控制 link 标签的 rel 属性，默认 `'preload'`，可改为 `'prefetch'`。
+ `imageDir`：指定图片目录，支持绝对路径或相对路径。

`preload` 和 `prefetch` 都是 `<link>` 标签的 `rel` 属性值，用于优化资源加载，但它们的作用和使用场景有明显区别：

---

#### 1. `preload`

+ **作用**：告诉浏览器“这个资源很快就会用到，请优先加载”。
+ **使用场景**：首屏渲染、关键资源（如首屏图片、字体、CSS、JS）。
+ **加载时机**：**立即并高优先级**加载。
+ **特点**：
  - 适合**当前页面渲染必须**的资源。
  - 资源会被立即下载，提升首屏速度。
  - 需要配合 `as` 属性指定资源类型。

---

#### 2. `prefetch`

+ **作用**：告诉浏览器“这个资源将来可能会用到，可以在空闲时提前下载”。
+ **使用场景**：二级页面、后续可能用到的资源（如路由懒加载 JS、下一个页面的图片）。
+ **加载时机**：**浏览器空闲时、低优先级**加载。
+ **特点**：
  - 适合**未来可能用到**但不是当前必须的资源。
  - 不会阻塞当前页面渲染。
  - 资源会被缓存，等真正需要时可直接使用。

---

#### 总结对比

| 属性     | 作用             | 加载时机         | 适用资源      | 优先级 |
| -------- | ---------------- | ---------------- | ------------- | ------ |
| preload  | 当前页面马上要用 | 立即，高优先级   | 首屏/关键资源 | 高     |
| prefetch | 未来可能会用     | 空闲时，低优先级 | 二级/后续资源 | 低     |


---

**一句话理解**：  

+ `preload` 是“马上要用，赶紧下！”  
+ `prefetch` 是“以后可能用，闲了再下。”

---

### 4. **使用示例**

```typescript
// vite.config.ts
import preloadImageVite from './plugins/preloadImageVite';

export default defineConfig({
  plugins: [
    preloadImageVite({
      rel: 'preload',
      imageDir: '../src/assets/image',
    }),
  ],
});
```

---

### 5. **注意事项**

+ 图片路径需与 `imageDir` 配置一致，否则无法正确生成 preload 标签。
+ 插件仅在构建时生效，开发时不会注入 preload 标签。

---

### 6. **优化点**

+ 支持更多图片格式。
+ 支持多目录扫描。
+ 支持自定义 publicPath。

---

### 7. **总结**

+ 插件通过 Vite 钩子实现图片预加载，提升页面加载速度。
+ 配置灵活，可根据需求调整 rel 和图片目录。
+ 适合生产环境使用，开发时无需预加载。

