---
title: "Extraxt all text from pptx files with python"
date: 2023-05-23T10:59:29+02:00
draft: false
tags: ["python"]
---

Enabling the features of current age open source LLMs on MS Office documents can be an efficiency game changer in many business tasks. The first step is to extract text from these documents before feeding the LLMs. The initial answer on how to do it from ChatGPT and You Chat was almost working, but there was missing text from grouped shapes. This code snipped does extract and does a little bit of cleanup:


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

    
def cleanup_text(text:str) -> str:
    # collapse duplicate spaces, remove non printable character
    return re.sub('\s+', ' ',re.sub(r"[^\x20-\x7E]", " ", text)).strip()
    
prs = Presentation("file.pptx")
notes_texts, slide_texts = [], []
for i_slide,slide in enumerate(prs.slides):
        # process slide notes
        notes_texts.append(
            "\n".join([shape.text for shape in slide.notes_slide.shapes if is_good_text(shape.text)])
            if slide.has_notes_slide else ""
        )

        slide_texts += [
            cleanup_text(text)
            for text in (
                [  # Text from ungrouped shapes
                    shape.text
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