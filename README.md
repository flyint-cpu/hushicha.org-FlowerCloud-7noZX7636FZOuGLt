
安装
pnpm install javascript\-obfuscator


安装之后 在项目根目录新建一个 obfuscator.js
![](https://img2024.cnblogs.com/blog/1617423/202411/1617423-20241114095856008-1213654786.png)


在 obfuscator.js 写入以下代码 直接复制粘贴
\`
/\*\*


* @用法
* vite打包完成后，使用命令行nodejs执行本文件: node obfuscator.js
* 它会挨个把里面的js文件做混淆然后替换
* * @说明
* 本质就是依赖这个工具
* 底层实现就是把代码全部作为一个字符串丢给它，它内部调用其他包来分析语法，做混淆替换
* * @doc [https://github.com/javascript\-obfuscator/javascript\-obfuscator](https://github.com)
* * @拓展
* obfuscator.js也有对应 webpack 的 plugin 和 rollup（vite打包用的就是rollup） 的 plugin
* 实现起来比较简单，如有需要也可以自己找符合要求的plugin或者自己写一个，本质上就是把这个文件的执行过程自动追加到打包过程中
\*/
const JavaScriptObfuscator \= require('javascript\-obfuscator')
const fs \= require('fs')


// 配置
const buildDir \= './dist/assets/'


/\*\*


* 获取目录下所有js文件及内容字符串
* @result {fileName:string, content:string}\[]
\*/
const getJsFileList \= (dir) \=\> new Promise((resolve) \=\> {
fs.readdir(dir, (err, files) \=\> {
if (err) return reject(`[obfuscator] output dir not exist!`)



```
 return resolve(Promise.all(files.filter(fileName => fileName.endsWith('.js')).map(fileName => new Promise(resolveInner => {
     fs.readFile(dir + fileName, (err, data) => {
         return resolveInner({ fileName, content: data.toString() })
     })
 }))))

```

})
})


getJsFileList(buildDir).then(list \=\> {
console.log(`[obfuscator] start`)
Promise.all(list.map(it \=\> new Promise(resolve \=\> {
console.log(it.fileName)



```
    const obfuscationResult = JavaScriptObfuscator.obfuscate(it.content, {
        /** 这些都是配置 */
        compact: false,
        controlFlowFlattening: true,
        controlFlowFlatteningThreshold: 1,
        numbersToExpressions: true,
        simplify: true,
        stringArrayShuffle: true,
        splitStrings: true,
        stringArrayThreshold: 1
    })
    fs.writeFile(buildDir + it.fileName, obfuscationResult.getObfuscatedCode(), () => {
        console.log(`[obfuscator] file done ${ it.fileName }`)
        resolve(1)
    })
}))).then(() => {
    console.log(`[obfuscator] finished`)
})

```

})
\`


目前只是配置好了要执行的脚本，如若要加密 先打包项目 执行pnpm build，打包结束后通过node命令执行脚本 node obfuscator.js。执行成功后会有提示
这时候dist目录下的assets文件夹内容已经加密完成。可直接部署到nginx。


常见错误：


问题描述
打包好项目发布上传到 nginx 后，浏览器访问，出现一下报错信息：


Failed to load module script: Expected a JavaScript module script but the server responded with a MIME type of "text/html". Strict MIME type checking is enforced for module scripts per HTML spec
这个错误提示的意思是：网页中使用了模块脚本（module script），但服务器返回的 MIME 类型是 “text/html”，而不是 JavaScript 类型。根据 HTML 规范，严格的 MIME 类型检查会对模块脚本进行强制执行。


将vite配置中 base 改为 '/',我的之前是 /。按照以下配置改完后重新打包部署，重启nginx即可解决
![](https://img2024.cnblogs.com/blog/1617423/202411/1617423-20241114100443154-637570932.png)
![](https://img2024.cnblogs.com/blog/1617423/202411/1617423-20241114100417995-416198001.png)


 本博客参考[FlowerCloud机场](https://hanlianfangzhi.com)。转载请注明出处！
