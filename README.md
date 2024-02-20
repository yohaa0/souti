 

#### 文章目录

*   *   [流程]
    *   [将excel表格数据解析为json格式]
    *   [实现答题]
    *   [测试地址]

### [](https://blog.csdn.net/qq_23857415/article/details/134778601)流程

首先将题库excel表格文件转为json格式的字符串保存到手机上  
然后在从这个文件读取json字符串转为json数组对象进行匹配查找控件答题

### [](https://blog.csdn.net/qq_23857415/article/details/134778601)将excel表格数据解析为json格式

```
/* 文件名 */
var SelectFilename = "知识答题题库.xls";
/* 下载路径 */
var FileRoute = "cloud/download/";
/* 下载链接 可以将表格文件上传到云空间，右键复制下载链接*/
var SelectFileListUrl = "http://cdn.smartcloudscript.com/package/%E7%9F%A5%E8%AF%86%E7%AD%94%E9%A2%98%E9%A2%98%E5%BA%93.xls"
utilsObj.downLoadFile(SelectFileListUrl, FileRoute + SelectFilename);

/* 解析excel数据 */
var excelData = readExcelData(FileRoute + SelectFilename, "题库");
console.log(JSON.stringify(excelData))
if (SelectFilename.indexOf(".") != -1) {
    var SelectFileName = SelectFilename.substring(0, SelectFilename.indexOf("."));
    files.write("/sdcard/" + FileRoute + SelectFileName + ".js", JSON.stringify(excelData));
    console.log("已将excel数据保存到手机：" + FileRoute + SelectFileName + ".js")
}

function readExcelData(fileRoute, SheetName) {
    var questionBankList = [];
    try {
        /* 获取到Excel文件 */
        var file = new File("/sdcard/" + fileRoute);
        var wb = Workbook.getWorkbook(file);
        /* 获取指定的sheet页码   通过指定的Sheet页的名字获取指定的Sheet页，也可以通过索引获取Sheet */
        var sheet = wb.getSheet(SheetName);
        /* 以第一行为键名，必须是中文或者是应为，不能用符合和数字 */
        var keyList = []
        for (var column = 0; column < sheet.getColumns(); column++) {
            var cell = sheet.getCell(column, 0);
            /* 去除第一行表格里的符号和空格后的内容作为键名 */
            var keyName = extractChineseAndEnglish(cell.getContents());
            if (keyName != "") {
                keyList.push(keyName);
            }
        }
        console.log(keyList)
        /* 循环获取指定的行和列的单元格的值     外循环控制行，内循环控制列 */
        for (var row = 0; row < sheet.getRows(); row++) {
            if (row > 0) {
                var obj = {};
                for (var column = 0; column < sheet.getColumns(); column++) {
                    /*  获取指定的单元格的数据  通过getCell方法获取指定单元格对象，参数是column,row,索引从0开始 */
                    cell = sheet.getCell(column, row);
                    if (column < keyList.length) {
                        obj[keyList[column]] = cell.getContents();
                    }
                }
                questionBankList.push(obj);
            }
        }
        /* 关闭表格 */
        wb.close();

    } catch (e) {
        console.log(e);
    }
    return questionBankList;
}

function extractChineseAndEnglish(str) {
    // 使用正则表达式匹配中文和英文
    var chinese = str.match(/[\u4e00-\u9fa5]+/g) || [];
    var english = str.match(/[a-zA-Z]+/g) || [];
    // 将匹配到的中文和英文连接成新的字符串
    var result = '';
    if (chinese.length > 0) {
        result += chinese.join('');
    }
    if (english.length > 0) {
        result += english.join('');
    }
    return result;
};

```

### [](https://blog.csdn.net/qq_23857415/article/details/134778601)实现答题

```
launchApp("云控");
sleep(2000)
launchApp("电网头条");
sleep(5000)

while (true) {

  var returned取消 = text("取消").className("android.widget.Button").findOne(100);
  if (returned取消) {
    click(returned取消.bounds().centerX() + random(-5, 5), returned取消.bounds().centerY() + random(-5, 5));
    sleep(500);
  } else {
    toastLog("未找到符合条件的取消控件");
  }
  var returned进入答题 = text("进入答题").className("android.widget.TextView").findOne(100);
  if (returned进入答题) {
    click(returned进入答题.bounds().centerX() + random(-5, 5), returned进入答题.bounds().centerY() + random(-5, 5));
    sleep(500);
  } else {
    toastLog("未找到符合条件的进入答题控件");
  }
  var returned_iv_question_parent = id("com.dianwang:id/iv_question_parent").className("android.widget.ImageView").findOne(100);
  if (returned_iv_question_parent) {
    click(returned_iv_question_parent.bounds().centerX() + random(-5, 5), returned_iv_question_parent.bounds().centerY() + random(-5, 5));
    sleep(500);
  } else {
    toastLog("未找到符合条件的每日答题控件");
  }
  var returned_tv_question_advise = id("com.dianwang:id/tv_question_advise").className("android.widget.TextView").findOne(100);
  if (returned_tv_question_advise) {
    click(returned_tv_question_advise.bounds().centerX() + random(-5, 5), returned_tv_question_advise.bounds().centerY() + random(-5, 5));
    sleep(500);
  } else {
    toastLog("未找到符合条件的直接答题控件");
  }
  var returnedtv_type = id("com.dianwang:id/tv_type").className("android.widget.TextView").findOne(100);
  if (returnedtv_type) {
    toastLog("已经进入答题");
    DeviceLog("已经进入答题", true, true);
    sleep(500);
    break;
  } else {
    toastLog("未找到符合条件的控件");
  }

}
var contentList = []

/* 文件在手机上的完整路径 */
var filePath = "/sdcard/cloud/download/知识答题题库.js";
var file = new java.io.File(filePath);

if (file.exists()) {
  var fileString = files.read(filePath);
  console.log(fileString);

  try {
    /* 字符串转json数组对象 */
    contentList = JSON.parse(fileString);

  } catch (e) {
    console.log("文件内容不是有效的 JSON 字符串");
  }
} else {
  console.log("文件不存在");
};

// console.log(contentList);

var i = 1;
while (true) {

  DeviceLog("第" + i + "次答题", true, true);

  每组答题();

  /* 判断i是否等于10 */
  if (i === 5) {
    /*break 跳出循环  */

    toastLog("已经做了" + i + "次答题");
    break;
  }

  /* 我们打印i的值 */
  console.log(i)
  /* 每循环一次，i就加1 */
  i++;
  var returned = id("com.dianwang:id/tv_accuracy").text("100%").className("android.widget.TextView").findOne(100);
  if (returned) {
    toastLog("结果正确");
    DeviceLog("结果正确", true, true);
    sleep(500);
    break;
  } else {
    toastLog("未找到符合条件的控件");
  }
  var returnedtv_again = id("com.dianwang:id/tv_again").className("android.widget.TextView").findOne(100);
  if (returnedtv_again) {
    click(returnedtv_again.bounds().centerX() + random(-5, 5), returnedtv_again.bounds().centerY() + random(-5, 5));
    sleep(500);
  } else {
    toastLog("未找到符合条件的再来一组控件");
  }

};
function 每组答题() {
  while (true) {
    var returned = id("com.dianwang:id/tv_type").className("android.widget.TextView").findOne(100);
    if (returned) {
      toastLog(returned.text());
      sleep(500);
      log(contentList.length)
      var contentListType = contentList.filter((item) => {
        return item["题型"].indexOf(returned.text()) != -1;
      });
      log(contentListType.length)
      if (contentListType.length > 0) {
        for (var i = 0, len = contentListType.length; i < len; i++) {
          console.log(i)
          var returnedtv_description = id("com.dianwang:id/tv_description").className("android.widget.TextView").findOne(100);
          if (returnedtv_description) {
            // toastLog(returnedtv_description.text());
            if (contentListType[i]["题干"].indexOf(returnedtv_description.text()) != -1) {
              toastLog("找到了题目");
              sleep(500);
              var 答案 = contentListType[i]["答案"]
              toastLog(答案);
              /* 循环点击答案 */
              for (var t = 0, lens = 答案.length; t < lens; t++) {
                /*循环体 */
                console.log(t)
                /* toastLog(答案[t]); */
                var returned = textContains(答案[t] + "、").classNameContains("android.widget.TextView").findOne(100);
                if (returned) {

                  click(returned.bounds().centerX() + random(-5, 5), returned.bounds().centerY() + random(-5, 5));
                  sleep(1500);
                } else {
                  toastLog("未找到符合条件的控件");
                }
                sleep(500);
              };
              var returned确定 = text("确定").className("android.widget.TextView").findOne(100);
              if (returned确定) {
                click(returned确定.bounds().centerX() + random(-5, 5), returned确定.bounds().centerY() + random(-5, 5));
                sleep(500);
              } else {
                toastLog("未找到符合条件的确定控件");
              }
              var returned下一题 = text("下一题").className("android.widget.TextView").findOne(100);
              if (returned下一题) {
                click(returned下一题.bounds().centerX() + random(-5, 5), returned下一题.bounds().centerY() + random(-5, 5));
                sleep(500);
              } else {
                toastLog("未找到符合条件的下一题控件");
              }
              break;
            }
          } else {
            toastLog("未找到符合条件的题目控件");
          }
          if (i === contentListType.length - 1) {
            toastLog("没有找到题");
            var returnedA = textContains("A、").classNameContains("android.widget.TextView").findOne(100);
            if (returnedA) {

              click(returnedA.bounds().centerX() + random(-5, 5), returnedA.bounds().centerY() + random(-5, 5));
              sleep(1500);
            } else {
              toastLog("未找到符合条件的A控件");
            }
            var returned确定 = text("确定").className("android.widget.TextView").findOne(100);
            if (returned确定) {
              click(returned确定.bounds().centerX() + random(-5, 5), returned确定.bounds().centerY() + random(-5, 5));
              sleep(500);
            } else {
              toastLog("未找到符合条件的确定控件");
            }
            var returned下一题 = text("下一题").className("android.widget.TextView").findOne(100);
            if (returned下一题) {
              click(returned下一题.bounds().centerX() + random(-5, 5), returned下一题.bounds().centerY() + random(-5, 5));
              sleep(500);
            } else {
              toastLog("未找到符合条件的下一题控件");
            }
          }
        };
      } else {
        toastLog("未找到符合条件的题库题型");
      }
    } else {
      toastLog("未找到符合条件的题型控件");
    }
    console.log(i)
    var returned = text("答题结果").className("android.widget.TextView").findOne(100);
    if (returned) {
      toastLog("找到答题结果控件");
      sleep(500);
      break;
    } else {
      toastLog("未找到符合条件的控件");
    }
  };
}

```

### [](https://blog.csdn.net/qq_23857415/article/details/134778601)测试地址

测试地址：http://smartcloudscript.com

![在这里插入图片描述](https://img-blog.csdnimg.cn/9f7f26ab381d41d994dcab62ddbd8672.gif#pic_center)

 

  

本文转自 [https://blog.csdn.net/qq\_23857415/article/details/134778601](https://blog.csdn.net/qq_23857415/article/details/134778601)，如有侵权，请联系删除。
