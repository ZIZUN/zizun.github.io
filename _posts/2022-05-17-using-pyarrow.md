---
title:  "[python] 대용량 데이터 처리 위한 pyarrow 사용법"
excerpt: "대용량 데이터 처리 위한 pyarrow 사용법"
toc: true
toc_sticky: true
header:
  teaser: /assets/images/logo.jpg

categories:
  - pytorch
tags:
  - pytorch

last_modified_at: 2021-08-23T09:06:00-05:00
---

- Make arrow example

```python
######################################################## Make arrow example
import pandas as pd
import pyarrow as pa

dataframe = pd.DataFrame(
    {
        'image_id': [1,2,3],
        'image': ['adsfasdff', 'adsfasdff', 'adsfasdff']        
    }
)

table = pa.Table.from_pandas(dataframe)

save_dir = '/home/leesm/Project/2022_1/WebQA/data/test_pyarrow/dataset.arrow'

with pa.OSFile(save_dir, "wb") as sink:
    with pa.RecordBatchFileWriter(sink, table.schema) as writer:
        writer.write_table(table)
########################################################   
```

- Read arrow example

```python
######################################################## Read arrow example
import pyarrow as pa

from PIL import Image
from io import BytesIO
import  base64

data_dir = '/home/leesm/Project/2022_1/WebQA/data/test_pyarrow/image.arrow'

table = pa.ipc.RecordBatchFileReader(
        pa.memory_map(data_dir, "r")
    ).read_all()
print(table['image_id'])
# print(table['image'][0])

im = Image.open(BytesIO(base64.b64decode(table['image'][0].as_py())))   

im.save('/home/leesm/Project/2022_1/WebQA/data/test_pyarrow/img_sample.png')
########################################################   
```

pytorch 사용자일 경우 __getitem__ 에서 아래와 같은 방식으로 작성해주면 된다.

```python
    def __getitem__(self, item):
        output = self.processed_dataset[item]
        
     
        im = Image.open(BytesIO(base64.b64decode(self.table['image'][self.imageid_to_arrowid[output['patch_images']]].as_py())))
        
        patch_img = self.patch_resize_transform(im)       
        output.update({'patch_images': patch_img})    
        
        return {key: torch.tensor(value) for key, value in output.items()}
```
