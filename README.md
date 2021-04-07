Open source docxtemplater image module
==========================================
This package is open source. There is also a [paid version](https://docxtemplater.com/modules/image/) maintained by docxtemplater author.

Note this version is compatible until docxtemplater 3.5.2. There are good forks of this repository.

Installation
=============
If not already done, install docxtemplater by following its [installation guide](https://docxtemplater.readthedocs.io/en/latest/installation.html).

Node.js: after installing docxtemplater, install the open source image module:
```
npm install open-docxtemplater-image-module
```

For the browser find builds in `build/` directory.

Alternatively, you can create your own build from the sources:
```
npm run compile
npm run browserify
npm run uglify
```

Usage
=====
Assuming your **docx** or **pptx** template contains only the text `{%image}`:
```javascript
//Node.js example
var ImageModule = require('open-docxtemplater-image-module');

//Below the options that will be passed to ImageModule instance
var opts = {}
opts.centered = false; //Set to true to always center images
opts.fileType = "docx"; //Or pptx

//Pass your image loader
opts.getImage = function(tagValue, tagName) {
    //tagValue is 'examples/image.png'
    //tagName is 'image'
    return fs.readFileSync(tagValue);
}

//Pass the function that return image size
opts.getSize = function(img, tagValue, tagName) {
    //img is the image returned by opts.getImage()
    //tagValue is 'examples/image.png'
    //tagName is 'image'
    //tip: you can use node module 'image-size' here
    return [150, 150];
}

var imageModule = new ImageModule(opts);

var zip = new JSZip(content);
var doc = new Docxtemplater()
    .attachModule(imageModule)
    .loadZip(zip)
    .setData({image: 'examples/image.png'})
    .render();

var buffer = doc
        .getZip()
        .generate({type:"nodebuffer"});

fs.writeFile("test.docx",buffer);
```

Some notes regarding templates:
* **docx** files: the placeholder `{%image}` must be in a dedicated paragraph.
* **pptx** files: the placeholder `{%image}` must be in a dedicated text cell.

Centering images
================
You can center all images by setting the global switch to true `opts.centered = true`.

If you would like to choose which images should be centered one by one:
* Set the global switch to false `opts.centered = false`.
* Use `{%image}` for images that shouldn't be centered.
* Use `{%%image}` for images that you would like to see centered.

In **pptx** generated documents, images are centered vertically and horizontally relative to the parent cell.
