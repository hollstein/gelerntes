---
title: "Which OCR to use today? (extracting text from images in power point)"
date: 2023-05-28T15:08:56+02:00
draft: false
tags: ["open source", "ocr", "python"]
---


Recently I added these comments to my prototype text extraction code from documents:

```python
import pytesseract  # this works really bad
pytesseract.pytesseract.tesseract_cmd = "/usr/bin/tesseract"

import easyocr  # looking much better
reader = easyocr.Reader(['en'],gpu=False)
```

The types of images I'm after are those found in typical Power Point slides in a business context. So mostly digital fonts and almost nothing handwritten. I gave both [tesseract](https://pypi.org/project/pytesseract/) and [easyocr](https://pypi.org/project/easyocr/) a try and for my use-case, easyocr is the clear winner.

It is as simple as that:


```python
ocr_method = ... # set to your choice
prs = Presentation('file.pptx')

def cleanup_text(text:str) -> str:
    return re.sub('\s+', ' ',re.sub(r"[^\x20-\x7E]", " ", text)).strip()

for i_slide,slide in enumerate(prs.slides):

    for i_shape,shape in enumerate(slide.shapes,start=i_shape):
        if shape.shape_type == MSO_SHAPE_TYPE.PICTURE:
            
            if ocr_method == "tesseract":
                try:
                    image_text = cleanup_text(
                        pytesseract.image_to_string(
                            Image.open(
                                io.BytesIO(
                                    shape.image.blob
                                )
                            )
                        )
                    )
                except TypeError:
                    image_text=""
                    
            elif ocr_method == "easyocr":
                
                if shape.image.ext in ["jpg", "jpeg", "png"]:
                    try:
                        image_text = " ".join(
                            [
                                cleanup_text(tt[1]) 
                                for tt in reader.readtext(shape.image.blob) if tt[2]>0.8
                            ]
                        )
                    except RuntimeError:
                        image_text = ""
                        print(f"OCR Error: {shape.image.ext}")
                else:
                    image_text = ""
                    
            else:
                raise ValueError()
```

