<img :src="new URL(../assets/images/${xxx}.png, import.meta.url).href" />
但这个对完全动态路径也可能受构建分析限制，综合上还是 import.meta.glob 最可靠。 为什么说会受分析限制
-------

因为 new URL(..., import.meta.url) 这类写法，Vite 需要在构建时“提前知道可能会用到哪些文件”，才能把图片拷贝、重命名（带 hash）并替换成最终 URL。

../assets/images/${xxx}.png 如果 xxx 完全动态（运行时才知道），构建器无法确定候选文件集合，就会出现图片找不到情况。
import.meta.glob 的本质是：你先声明一个“可选文件集合”，让构建器能分析到，再在运行时从集合里按 key 取，所以更稳。