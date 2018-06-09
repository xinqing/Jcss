# js实现csv和excel导出

在html5中，DOM给文件中添加了一个files集合,在选取文件中，files中包含一个File对象，每个对象都有下列属性： 
- name: 本地文件的文件名
- size：文件的大小
- type: 字符串，文件的MIME类型
- lastModifiedDate: 字符串，文件上一次被修改的时间

同时，使用FileReader对象，web应用程序可以异步的读取存储在用户计算机上的文件(或者原始数据缓冲)内容，可以使用File对象或者Blob对象来指定所要处理的文件或数据。FileReader 提供了如下 几个方法。 
- readAsText(file,encoding)：以纯文本形式读取文件，将读取到的文本保存在 result 属 性中。第二个参数用于指定编码类型，是可选的。
- readAsDataURL(file)：读取文件并将文件以数据 URI的形式保存在 result 属性中。  
- readAsBinaryString(file)：读取文件并将一个字符串保存在 result 属性中，字符串中的 每个字符表示一字节。   
- readAsArrayBuffer(file)：读取文件并将一个包含文件内容的 ArrayBuffer 保存在 result 属性中。 这些读取文件的方法为灵活地处理文件数据提供了极大便利。例如，可以读取图像文件并将其保存 为数据 URI，以便将其显示给用户，或者为了解析方便，可以将文件读取为文本形式。 

## 先来个例子

需求：若读取文件为图片，则以img展示出来，其他情况则以text的形式输出。

 
1. 若要读取文件，首先需要一个input元素  

```html
<input type="file" id="files-list">
```
2. 接下来，若读取的文件为图片，使用readAsDataURL将其转化为图片，否则转化为text。 相应的转化结果会保存在result中。


```js
reader = new FileReader();
if (/image/.test(files[0].type)) {
    // 选择文件类型为图片
    reader.readAsDataURL(files[0]);
    type = "image";
} else {
    // 其他文件类型，并指定编码类型
    reader.readAsText(files[0], 'gb2312');
    type = "text";
}

```

3. 读取完成后，将图片利用img标签展示出来；若为text则挂载在页面上

```js
reader.onload = function () {
    var html = "";
    switch (type) {
        case "image":
            html = "<img src=\"" + reader.result + "\">";
            break;
        case "text":
            html = reader.result;
            console.log(html);
            break;
    }
    output.innerHTML = html;
};
```
[演示链接](https://hecun0000.github.io/Jcss/file/readFile.html)    

**读取图片**   

![image](http://oxi9lrcsm.bkt.clouddn.com/IMG.png)

**读取csv文件**
![image](http://oxi9lrcsm.bkt.clouddn.com/csv.png)

    在利用readAsText进行读取的时候，要指定一下编码形式。如readAsText(files[0], 'gb2312');   
    
## csv的导入 

    csv文件的特点： 每一行用换行符进行分隔，每一行的数据中每一项利用逗号分隔。

上面已经可以将csv作为text进行输出了。下来继续拿取数据。

利用上面所说的csv的特点就可以利用js进行循环遍历拿到每一项的数据。  

[演示链接](https://hecun0000.github.io/Jcss/file/cvsExport.html)   

其中，我将CVS中的数据转化后，并拼在了table元素中：

```js
// 将读取的数据转化为table
function textToCsv(data) {
    var allRows = data.split(/\n/);
    var table = '<table>';
    for (var singleRow = 0; singleRow < allRows.length - 1; singleRow++) {
        if (singleRow === 0) {
            table += '<thead>';
            table += '<tr>';
        } else {
            table += '<tr>';
        }
        var rowCells = allRows[singleRow].split(',');
        for (var rowCell = 0; rowCell < rowCells.length; rowCell++) {
            if (singleRow === 0) {
                // 表格的标题
                table += '<th>';
                table += rowCells[rowCell];
                table += '</th>';
            } else {
                // 表格内容
                table += '<td>';
                table += rowCells[rowCell];
                table += '</td>';
            }
        }
        if (singleRow === 0) {
            table += '</tr>';
            table += '</thead>';
            table += '<tbody>';
        } else {
            table += '</tr>';
        }
    }
    table += '</tbody>';
    table += '</table>';
    console.log(table);
    document.getElementById('table').innerHTML = table;
}
```
**效果图如下：**

![image](http://oxi9lrcsm.bkt.clouddn.com/csvout.png)

## 既然都读取出来，下载？  

关于下载的操作，可以借助于a标签。先介绍下相关的属性： 
- download: 用于指定下载的文件名
- href：为所要下载的文件的链接（需为本地链接）

上一个列子的下载就是用的上面的做法： 

```html
<a href="./file/会员.csv" download="demo.csv">下载测试文件</a>
```

先准备数据
csv文件的头部数据：


```js
var head = [
    ['姓名（最多10个字）', '手机号（必填）', '等级', '生日（比如：1989/08/08 或 1989-08-08）', '积分（限整数）']
];
```

csv文件的主体数据：

```js
var people = [{
    "name": "吴三桂",
    "phone": "18709237410",
    "level": "黄金",
    "birthday": "1989/8/5",
    "points": "100"
}, {
    "name": "史泰龙",
    "phone": "18709237401",
    "level": "青铜2",
    "birthday": "1993/9/6",
    "points": "300"
}, {
    "name": "阿超",
    "phone": "18883920287",
    "level": "白金",
    "birthday": "1993/9/3",
    "points": "500"
}];
```

下来就是数据的格式转换： 

```js
//. 将数据push到大数组中 
var p = people;
for (var i = 0; i < p.length; i++) {
    head.push([p[i].name, p[i].phone, p[i].level, p[i].birthday, p[i].points]);
}
// 按照csv文件内容格式，把每个数组用 , 连接，形成一行，并存入新数组 
var csvRows = [];
for (var j = 0; j < head.length; j++) {
    csvRows.push(head[j].join(','))
}
//s4. 把新数组用 \n 回车连接 
var csvString = csvRows.join('\n'); 
```

最后实现下载： 

```js
//创建a标签
var a = document.createElement('a');
a.setAttribute("href", 'data:attachment/csv,' + encodeURI(csvString));
a.setAttribute("target", '_blank');
a.setAttribute("download", '会员.csv');
// 插入到HTML 中，点击实现下载，然后移除元素
document.querySelector('.demo').appendChild(a);
a.click();
document.querySelector('.demo').removeChild(a);
```
[演示链接](https://hecun0000.github.io/Jcss/file/csvDownload.html)  
**效果图**
![image](http://oxi9lrcsm.bkt.clouddn.com/csv12321.png)

另外，
将json文件转化为csv格式可以利用[json2csv](https://github.com/zemirco/json2csv)



下面将使用[js-xlsx](https://docs.sheetjs.com/#sheetjs-js-xlsx)对csv文件进行操作：

## excel文件读取  

xlsx相关api:  
- XLSX.utils.sheet_to_json方法解析表格对象返回相应的JSON数据
- XLSX.read()以二进制流方式读取excel数据

在JavaScript中，有2个函数分别用来处理解码和编码base64 字符串：

- atob() 函数能够解码通过base-64编码的字符串数据。
- btoa() 函数能够从二进制数据“字符串”创建一个base-64编码的ASCII字符串。

步骤：   
1. readAsArrayBuffer可以将读取的数据转化为二进制数据；
2. 通过btoa()将数据转化为base64格式；
3. 然后结合XLSX.utils.sheet_to_json转化为json

```js
var wb;//读取完成的数据
 //导入
function importf(obj) {
    if (!obj.files) {
        return;
    }
    var f = obj.files[0];
    var reader = new FileReader();
    reader.readAsArrayBuffer(f);
    reader.onload = function (e) {
        var data = e.target.result;
        var wb =XLSX.read(btoa(fixedData(data)), { type: 'base64' });//将数据转化为二进制
        //wb.SheetNames[0]是获取Sheets中第一个Sheet的名字
        //wb.Sheets[Sheet名]获取第一个Sheet的数据
        document.getElementById("excel").innerHTML = JSON.stringify(XLSX.utils.sheet_to_json(wb.Sheets[wb.SheetNames[0]]));
    };

}
```
```js
//文件流转BinaryString
function fixedData(data) {
    let o = ''
    let l = 0
    const w = 10240
    for (; l < data.byteLength / w; ++l) o += String.fromCharCode.apply(null, new Uint8Array(data.slice(l * w,
        l * w + w)))
    o += String.fromCharCode.apply(null, new Uint8Array(data.slice(l * w)))
    return o
}
```

效果图预览： 
![image](http://oxi9lrcsm.bkt.clouddn.com/xlsxsave.png)
[查看演示](https://hecun0000.github.io/Jcss/file/xlsx-export.html)

附上一个vue-element-admin关于Excel文件读取的源码：
[查看链接](https://github.com/PanJiaChen/vue-element-admin/blob/master/src/components/UploadExcel/index.vue)


## excel文件导出












参考文章： 

1. [理解DOMString、Document、FormData、Blob、File、ArrayBuffer数据类型--张鑫旭](http://www.zhangxinxu.com/wordpress/2013/10/understand-domstring-document-formdata-blob-file-arraybuffer/)  
2. [文件和二进制数据的操作--阮一峰](http://javascript.ruanyifeng.com/htmlapi/file.html)