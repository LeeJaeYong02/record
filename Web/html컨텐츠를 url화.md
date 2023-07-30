# html컨텐츠를 url화

```
window.addEventListener('DOMContentLoaded', function() {
  var htmlContent = new Array(document.documentElement.innerHTML);
  var bl = new Blob(htmlContent, { type: 'text/html' });
  var url = URL.createObjectURL(bl);

  var iFrame = document.createElement("IFRAME");
  iFrame.setAttribute("src", url);
  document.body.appendChild(iFrame);
});
```
