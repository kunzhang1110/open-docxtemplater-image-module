# Open source docxtemplater image module 2

This package is open source. There is also a [paid version](https://docxtemplater.com/modules/image/) maintained by docxtemplater author.

This version is a fork from [open-docxtemplater-image-module](https://github.com/MaxRcd/open-docxtemplater-image-module) which solved some issues to make it compatible with docxtemplater 3.48.0 - 3.50.0, with limited functionality comparing to the paid version.

# Installation

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

# Usage
### Node

Assuming your **docx** or **pptx** template contains only the text `{%image}`:

```javascript
//Below the options that will be passed to ImageModule instance
var opts = {};
opts.centered = false; //Set to true to always center images
opts.fileType = "docx"; //Or pptx

//Pass your image loader
opts.getImage = function (tagValue, tagName) {
  //tagValue is 'examples/image.png'
  //tagName is 'image'
  const image = fs.readFileSync(tagValue);
  return image;
};

//Pass the function that return image size
opts.getSize = function (img, tagValue, tagName) {
  //tagValue is 'examples/image.png'
  //tagName is 'image'
  return [150, 150];
};

let docxFile;
try {
  docxFile = fs.readFileSync("examples/imageExample.docx");
  console.log("File read successfully!");
  // You can now work with the docxFile buffer, e.g., save it, send it over HTTP, etc.
} catch (error) {
  console.error("Error reading file:", error);
}

var imageModule = new ImageModule(opts);

// var zip = new JSZip(docxFile);
var zip = new PizZip(docxFile);

var doc = new Docxtemplater()
  .attachModule(imageModule)
  .loadZip(zip)
  .setData({ image: "examples/image.png" })
  .render();

var buffer = doc.getZip().generate({ type: "nodebuffer" });

fs.writeFile("examples/output.docx", buffer, (err) => {
  if (err) {
    console.error("Error writing file:", err);
  } else {
    console.log("File written successfully!");
  }
});
```

### Angular

```javascript
import { Component } from '@angular/core';
import Docxtemplater from 'docxtemplater';
import ImageModule from 'open-docxtemplater-image-module';
import * as JSZip from 'jszip';
import { HttpClient } from '@angular/common/http';
import * as saveAs from 'file-saver';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.scss'],
})
export class AppComponent {
  constructor(private http: HttpClient) {}

  loadFile(url: string): Promise<ArrayBuffer | undefined> {
    return this.http.get(url, { responseType: 'arraybuffer' }).toPromise();
  }

  async generateDocument() {
    const docxFile = await this.loadFile('/assets/imageExample.docx');
    const image = await this.loadFile('/assets/image.png');
    const image2 = await this.loadFile('/assets/image2.png');

    const images = [image, image2];
    let index = 0;

    const imageOpts = {
      getImage: (tagValue: string) => {
        const returnImage = images[index];
        index++;
        return returnImage;
      },
      getSize: (img: ArrayBuffer) => {
        // Default size of the image
        return [150, 150];
      },
    };

    const imageModule = new ImageModule(imageOpts);

    var zip = new JSZip(docxFile);
    var doc = new Docxtemplater()
      .loadZip(zip)
      .setData({ image: 'assets/image.png' })
      .attachModule(imageModule)
      .render();
    const blob = doc.getZip().generate({ type: 'blob' });
    saveAs(blob, 'output.docx');
  }
}
```

Some notes regarding templates:

- **docx** files: the placeholder `{%image}` must be in a dedicated paragraph.
- **pptx** files: the placeholder `{%image}` must be in a dedicated text cell.

# Centering images

You can center all images by setting the global switch to true `opts.centered = true`.

If you would like to choose which images should be centered one by one:

- Set the global switch to false `opts.centered = false`.
- Use `{%image}` for images that shouldn't be centered.
- Use `{%%image}` for images that you would like to see centered.

In **pptx** generated documents, images are centered vertically and horizontally relative to the parent cell.
