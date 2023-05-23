---
title: "Extraxt all text from pptx files with python"
date: 2023-05-23T10:59:29+02:00
draft: false
tags: ["python"]
---


Enabling the feats of current age open source LLMs on MS office documents can be a efficiency game changer in many business tasks. First step is to extract text from these documents before feeding the LLMs. The initial answer on how to do it from ChatGPT and You Chat was right, but only on the surface level. There I was missing text from grouped shapes. This code snipped does extract and does a little bit of cleanup:


```python
import re
from pptx import Presentation
from pptx.enum.shapes import MSO_SHAPE_TYPE

def is_good_text(text):
    # adapt to what makes sense
    if len(text) < 3:
        return False
    else:
        return True

prs = Presentation("file.pptx")
notes_texts, slide_texts = [], []
for i_slide,slide in enumerate(prs.slides):
        # process slide notes
        notes_text.append(
            "\n".join([shape.text for shape in slide.notes_slide.shapes if is_good_text(shape.text)])
            if slide.has_notes_slide else ""
        )

        slide_texts += [
            bytes(re.sub('\s+', ' ', text), 'utf-8').decode('utf-8', 'ignore')  # text cleanup
            for text in (
                [  # Text from ungrouped shapes
                    re.sub('\s+', ' ', shape.text)
                    for shape in slide.shapes 
                    if hasattr(shape, 'text') 
                    and is_good_text(shape.text)
                ] + [# Text from grouped shapes
                    shape.text
                    for group_shape in slide.shapes 
                    if group_shape.shape_type == MSO_SHAPE_TYPE.GROUP
                    for shape in group_shape.shapes
                    if shape.has_text_frame
                    and is_good_text(shape.text)
                ]    
            )
        ]
```