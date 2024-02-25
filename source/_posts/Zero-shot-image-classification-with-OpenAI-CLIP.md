---
language: en
title: "Zero-shot image classification with OpenAI's CLIP"
date: 2024-02-25 21:00:00
excerpt: "In this post, we will explore OpenAI's CLIP, a model that can understand images and texts, and use it to perform zero-shot image classification on the Animal Faces dataset."
index_img: /2024/02/25/Zero-shot-image-classification-with-OpenAI-CLIP/CLIP.png
banner_img: /2024/02/25/Zero-shot-image-classification-with-OpenAI-CLIP/CLIP.png
tags:
- Artificial Intelligence
- Machine Learning
- Computer Vision
- OpenAI
- CLIP
- Zero-shot learning
- Image classification
- Vision Transformer
---

In recent years, there is a trend in the field of Deep Learning to develop models that can understand multiple modalities, such as texts, images, sounds and videos. Google's Gemini is the latest example of this trend. These models open up new possibilities for AI applications, such as zero-shot learning, where the model can perform tasks without any training data. In this post, we will explore OpenAI's CLIP, a model that can understand images and texts, and use it to perform zero-shot image classification on the Animal Faces dataset.

# Introduction to CLIP

CLIP (Contrastive Language-Image Pre-Training) is a model developed by OpenAI that can understand images and texts. It is based on a Vision Transformer (ViT) and a Transformer-based language model. The model is trained to predict which of the image-text pairs in a batch is the correct pair. The model is trained on a large dataset of images and texts, and it learns to understand the relationship between the two modalities.

Here is the model architecture from the paper:

![CLIP model architecture](CLIP.png)

# The Animal Faces dataset

The Animal Faces dataset also known as Animal Faces-HQ (AFHQ), consists of 16,130 high-quality images at 512×512 resolution.

There are three domains of classes, each providing about 5000 images. By having multiple (three) domains and diverse images of various breeds per each domain, AFHQ sets a challenging image-to-image translation problem. The classes are:
- Cat
- Dog
- Wildlife

Link to the dataset: [Animal Faces-HQ](https://www.kaggle.com/datasets/andrewmvd/animal-faces)

# Loading the CLIP model

```python
import torch
from transformers import CLIPProcessor, CLIPModel

device = "cuda" if torch.cuda.is_available() else "cpu"
processor = CLIPProcessor.from_pretrained("openai/clip-vit-base-patch32")
model = CLIPModel.from_pretrained("openai/clip-vit-base-patch32").to(device)
```

# Zero-shot image classification

Download the Animal Faces dataset from the link provided above and extract it. The dataset is organized into three folders, one for each class. We will use the `glob` library to get the paths of all the images in the dataset.

```python
from PIL import Image
from glob import glob
from tqdm import tqdm
from sklearn.metrics import classification_report

classes = ["dog", "cat", "wild"]
text = [
  ("a photo of a dog", 0),
  ("a photo of a cat", 1),
  ("a photo of a wild animal", 2),
]


def classify_image(image_path):
  image = Image.open(image_path)
  inputs = processor(text=[t[0] for t in text], images=image, return_tensors="pt", padding=True).to(device)

  outputs = model(**inputs)
  logits_per_image = outputs.logits_per_image
  probs = logits_per_image.softmax(dim=1)
  return probs


image_paths = []
true_label = []
predicted_label = []
for cls in classes:
  for image_path in tqdm(glob(f"afhq/val/{cls}/*.jpg")):
    image_paths.append(image_path)
    true_label.append(cls)

    probs = classify_image(image_path)
    confidence, class_index = torch.max(probs, 1)

    predicted_label.append(classes[text[class_index][1]])

print(classification_report(true_label, predicted_label))
```

Run the above code give us the following result:

```
              precision    recall  f1-score   support

         cat       0.76      1.00      0.86       500
         dog       0.91      1.00      0.95       500
        wild       1.00      0.58      0.73       500

    accuracy                           0.86      1500
   macro avg       0.89      0.86      0.85      1500
weighted avg       0.89      0.86      0.85      1500
```

Nice! We achieved an accuracy of 86% by just using texts to classify images. This is the power of zero-shot learning.

But the 58% recall for the "wild" class is not good. Let's see some examples of misclassified images.

```python
import pandas as pd
from PIL import Image
from IPython.display import display

df = pd.DataFrame({
  "image_path": image_paths,
  "true_label": true_label,
  "predicted_label": predicted_label
})
wrong_df = df[df['true_label'] != df['predicted_label']]

sample = wrong_df.sample(1)
image_path = sample['image_path'].values[0]
true_label = sample['true_label'].values[0]
predicted_label = sample['predicted_label'].values[0]
image = Image.open(image_path)
display(image)
print(f'True label: {true_label}')
print(f'Predicted label: {predicted_label}')
```

![True label: wild. Predicted label: cat](output1.png)
![True label: wild. Predicted label: dog](output2.png)
![True label: wild. Predicted label: cat](output3.png)

As we can see, the model is misclassifying images of wild animals as cats and dogs. These animals, such as lions, wolves, and tigers, share some visual similarities with cats and dogs, which makes it difficult for the model to distinguish them.

One way to improve the model's performance is to change the text descriptions to include more specific details about the animals. For example, instead of "a photo of a wild animal", we can use "a photo of a lion" or "a photo of a tiger". This will help the model to better understand the differences between the classes.

Change the text descriptions as follows:

```python
text = [
  ("a photo of a dog", 0),
  ("a photo of a cat", 1),
  ("a photo of a wild animal", 2),
  ("a photo of a wolf", 2),
  ("a photo of a tiger", 2),
  ("a photo of a lion", 2),
  ("a photo of a fox", 2),
  ("a photo of a leopard", 2),
]
```

Run the code again and we get the following result:

```
              precision    recall  f1-score   support

         cat       1.00      1.00      1.00       500
         dog       1.00      0.94      0.97       500
        wild       0.94      1.00      0.97       500

    accuracy                           0.98      1500
   macro avg       0.98      0.98      0.98      1500
weighted avg       0.98      0.98      0.98      1500
```

Wow! We achieved an accuracy of 98% and improved the recall for the "wild" class to 100%. This is a significant improvement over the previous result.

# Conclusion

In this post, we explored OpenAI's CLIP, a model that can understand images and texts, and used it to perform zero-shot image classification on the Animal Faces dataset. We achieved an accuracy of 86% by just using texts to classify images. We also improved the model's performance by changing the text descriptions to include more specific details about the animals, and achieved an accuracy of 98% with 100% recall for the "wild" class. This demonstrates the power of zero-shot learning and the potential of models like CLIP to perform complex tasks without any training data.
