
## 获取用户的媒体数据  
通过使用这样的 getUserMedia 方法来获取数据源，这个 API 方法在不同浏览器中的实现方式不同，有的或许有前缀，这里用的是标准形式它接受一个配置对象和一个回调函数，其中回调函数接受一个stream类型的参数，使用方式如下  
```js  
getUserMedia({ video: false, audio: true }, function(stream) {
  // Now our stream does not contain any video!
});
```
  





