重点

服务器输出某一个 .docx 时把 buffer 转成 base64
···
fs.readFile(pathToFile, function(err, data) {
  var fileData = new Buffer(data).toString('base64');
  res.send(fileData);
});
···

插件中得到从服务器 get 的数据后可以直接通过 insertFileFromBase64 插入到正文中
```
  bodyObject.insertFileFromBase64(base64File, insertLocation);
```
> https://dev.office.com/reference/add-ins/word/body

直接用官网的例子可以得到当前打开的 docx 内容,
```
function getDocumentAsCompressed() {
  Office.context.document.getFileAsync(Office.FileType.Compressed, { sliceSize: 65536 /*64 KB*/ },
      function (result) {
          if (result.status == "succeeded") {
          // If the getFileAsync call succeeded, then
          // result.value will return a valid File Object.
          var myFile = result.value;
          var sliceCount = myFile.sliceCount;
          var slicesReceived = 0, gotAllSlices = true, docdataSlices = [];
          app.showNotification("File size:" + myFile.size + " #Slices: " + sliceCount);

          // Get the file slices.
          getSliceAsync(myFile, 0, sliceCount, gotAllSlices, docdataSlices, slicesReceived);
          }
          else {
          app.showNotification("Error:", result.error.message);
          }
  });
}

function getSliceAsync(file, nextSlice, sliceCount, gotAllSlices, docdataSlices, slicesReceived) {
  file.getSliceAsync(nextSlice, function (sliceResult) {
      if (sliceResult.status == "succeeded") {
          if (!gotAllSlices) { // Failed to get all slices, no need to continue.
              return;
          }

          // Got one slice, store it in a temporary array.
          // (Or you can do something else, such as
          // send it to a third-party server.)
          docdataSlices[sliceResult.value.index] = sliceResult.value.data;
          if (++slicesReceived == sliceCount) {
             // All slices have been received.
             file.closeAsync();
             onGotAllSlices(docdataSlices);
          }
          else {
              getSliceAsync(file, ++nextSlice, sliceCount, gotAllSlices, docdataSlices, slicesReceived);
          }
      }
          else {
              gotAllSlices = false;
              file.closeAsync();
              app.showNotification("getSliceAsync Error:", sliceResult.error.message);
          }
  });
}

function onGotAllSlices(docdataSlices) {
  var docdata = [];
  for (var i = 0; i < docdataSlices.length; i++) {
      docdata = docdata.concat(docdataSlices[i]);
  }

  var fileContent = new String();
  for (var j = 0; j < docdata.length; j++) {
      fileContent += String.fromCharCode(docdata[j]);
  }

  // Now all the file content is stored in 'fileContent' variable,
  // you can do something with it, such as print, fax...
}
```
> https://dev.office.com/reference/add-ins/shared/document.getfileasync

执行一下 getDocumentAsCompressed 可以在 onGotAllSlices 得到文件所有内容
ps 可以用 btoa 把内容转成 base64 发给服务器存储
```
  var fileData = Buffer.from(req.body.file, 'base64');
  fs.writeFile(pathToFile, fileData, function (err) {
    res.sendStatus(200);
  });
```
