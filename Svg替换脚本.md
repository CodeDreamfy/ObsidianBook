
好的，以下是一个基于Node.js的脚本，可以批量处理 SVG 文件中的 `fill` 属性和 `g` 标签的属性，将它们全部替换成 `currentColor`：

```js
const fs = require('fs');
const path = require('path');
const xml2js = require('xml2js');

// 读取指定目录下的所有 SVG 文件并处理
const svgDir = './svg';
fs.readdir(svgDir, (err, files) => {
  if (err) throw err;

  files.forEach(file => {
    if (path.extname(file) === '.svg') {
      const filePath = path.join(svgDir, file);

      // 读取 SVG 文件内容
      fs.readFile(filePath, 'utf8', (err, data) => {
        if (err) throw err;

        // 将 SVG 内容解析为 JSON 对象
        xml2js.parseString(data, (err, result) => {
          if (err) throw err;

          // 递归遍历解析后的 JSON 对象，替换 fill 属性和 g 标签的属性
          replaceFillAndGAttrs(result);

          // 将修改后的 JSON 对象转换回 SVG 格式的字符串
          const builder = new xml2js.Builder();
          const svgStr = builder.buildObject(result);

          // 将修改后的 SVG 内容写回到文件中
          fs.writeFile(filePath, svgStr, 'utf8', err => {
            if (err) throw err;
            console.log(`Updated ${filePath}`);
          });
        });
      });
    }
  });
});

// 递归遍历 JSON 对象，替换 fill 属性和 g 标签的属性
function replaceFillAndGAttrs(obj) {
  if (typeof obj === 'object') {
    // 处理 fill 属性
    if (obj.hasOwnProperty('$') && obj['$'].fill) {
      obj['$'].fill = 'currentColor';
    }

    // 处理 g 标签的属性
    if (obj.hasOwnProperty('g')) {
      obj['g'].forEach(gObj => {
        replaceFillAndGAttrs(gObj);
        if (gObj.hasOwnProperty('$') && gObj['$'].fill) {
          gObj['$'].fill = 'currentColor';
        }
      });
    }

    // 递归处理子元素
    Object.keys(obj).forEach(key => {
      if (key !== '$') {
        obj[key].forEach(childObj => {
          replaceFillAndGAttrs(childObj);
        });
      }
    });
  }
}
```

在上面的脚本中，我们使用了 `fs` 和 `path` 模块来读取和写入文件，使用了 `xml2js` 模块将 SVG 内容解析为 JSON 对象，并将修改后的 JSON 对象转换回 SVG 格式的字符串。

脚本会递归遍历 JSON 对象，找到所有包含 `fill` 属性的元素和 `g` 标签，将它们的 `fill` 属性全部替换成 `currentColor`。

你可以将上面的脚本保存为一个名为 `batch-process-svgs.js` 的文件，然后在命令行中执行 `node batch-process-svgs.js` 命令来运行它。注意，执行脚本前请备份好需要处理的 SVG 文件，以免意外修改导致不可逆的损失。