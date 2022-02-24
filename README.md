
# Quickstart: Extract printed text (OCR) using the Computer Vision REST API and Python

> [!NOTE]
> If you're extracting English language text, consider using the new [Read operation](../concept-recognizing-text.md). A [Python quickstart](./python-hand-text.md) is available. 

In this quickstart, you will extract printed text with optical character recognition (OCR) from an image using the Computer Vision REST API. With the [OCR](https://westcentralus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-1-ga/operations/56f91f2e778daf14a499f20d) method, you can detect printed text in an image and extract recognized characters into a machine-usable character stream.

You can run this quickstart in a step-by step fashion using a Jupyter Notebook.


## Prerequisites

* An Azure subscription - [Create one for free](https://azure.microsoft.com/free/cognitive-services/)
* You must have [Python](https://www.python.org/downloads/) installed if you want to run the sample locally.
* Once you have your Azure subscription, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesComputerVision"  title="Create a Computer Vision resource"  target="_blank">create a Computer Vision resource <span class="docon docon-navigate-external x-hidden-focus"></span></a> in the Azure portal to get your key and endpoint. After it deploys, click **Go to resource**.
    * You will need the key and endpoint from the resource you create to connect your application to the Computer Vision service. You'll paste your key and endpoint into the code below later in the quickstart.
    * You can use the free pricing tier (`F0`) to try the service, and upgrade later to a paid tier for production.
* [Create environment variables](../../cognitive-services-apis-create-account.md#configure-an-environment-variable-for-authentication) for the key and endpoint URL, named `COMPUTER_VISION_SUBSCRIPTION_KEY` and `COMPUTER_VISION_ENDPOINT`, respectively.

## Create and run the sample

To create and run the sample, do the following steps:

1. Copy the following code into a text editor.
1. Optionally, replace the value of `image_url` with the URL of a different image from which you want to extract printed text.
1. Save the code as a file with an `.py` extension. For example, `get-printed-text.py`.
1. Open a command prompt window.
1. At the prompt, use the `python` command to run the sample. For example, `python get-printed-text.py`.

```python

from azure.cognitiveservices.vision.customvision.training import CustomVisionTrainingClient
from azure.cognitiveservices.vision.customvision.prediction import CustomVisionPredictionClient
from azure.cognitiveservices.vision.customvision.training.models import ImageFileCreateBatch, ImageFileCreateEntry, Region
from msrest.authentication import ApiKeyCredentials
import os, time, uuid
import configparser
import os

import matplotlib.pyplot as plt
import numpy as np
import requests
from matplotlib.patches import Polygon

import cv2
import imutils

target_image_path= "<local-path-to-image>"
data = open(target_image_path, 'rb').read()

img = cv2.imdecode(np.array(bytearray(data), dtype='uint8'), 
    cv2.IMREAD_COLOR)
crop_bytes =bytes(cv2.imencode('.jpg', img)[1])

 # make a call to the text_recognition_url
text_recognition_url = "https://<name-of-resource>.cognitiveservices.azure.com/vision/v3.2/read/analyze"
response = requests.post(
    url=text_recognition_url, 
    data=crop_bytes, 
    headers={
        'Ocp-Apim-Subscription-Key': "<subscription-key>", 
        'Content-Type': 'application/octet-stream'})

response.raise_for_status()
operation_url = response.headers["Operation-Location"]

 # The recognized text isn't immediately available, so poll to wait for completion.
analysis = {}
poll = True
while (poll):
    response_final = requests.get(
        response.headers["Operation-Location"], 
        headers={'Ocp-Apim-Subscription-Key': "<subscription-key>"})
    analysis = response_final.json()
    time.sleep(1)
    if ("status" in analysis and analysis['status'] == 'succeeded'):
        poll = False
    if ("recognitionResults" in analysis):
        poll = False
    if ("status" in analysis and analysis['status'] == 'Failed'):
        poll = False       

#print(analysis)
result = analysis['analyzeResult']['readResults'][0]['lines']

for line in result:
    print(line['text'])
```

## Next steps

Next, explore a Python application that uses Computer Vision to perform optical character recognition (OCR); create smart-cropped thumbnails; and detect, categorize, tag, and describe visual features in images.

* [Computer Vision API Python Tutorial](https://github.com/Microsoft/Cognitive-Vision-Python)

* To rapidly experiment with the Computer Vision API, try the [Open API testing console](https://westcentralus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-1-ga/operations/56f91f2e778daf14a499f21b/console).
