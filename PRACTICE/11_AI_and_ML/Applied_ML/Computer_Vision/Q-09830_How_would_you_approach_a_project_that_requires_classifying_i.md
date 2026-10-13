---
type: "practice"
difficulty: "Deep"
---

# How would you approach a project that requires classifying images into categories the model has never seen during training?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Data vs Model Improvements](<../../../../11_AI_and_ML/Applied_ML/Computer_Vision/Data_vs_Model_Improvements.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To classify images into categories unseen during training, we need to leverage transfer learning and domain adaptation techniques effectively. The core idea is to use a pre-trained model as a starting point and adapt it to new classes with minimal data. This approach allows us to benefit from the pre-trained model's learned features while fine-tuning for specific tasks.

#### Triage

The question asks how to handle image classification in scenarios where the categories are unknown at training time. The key aspects include:

Transfer Learning: Utilizing a pre-trained model.
Domain Adaptation: Adapting the model to new domains or classes.
Data Augmentation and Labeling: Preparing data for fine-tuning.

#### First-Principles Solution
Pre-training Phase:
Use a large, publicly available dataset like ImageNet to pre-train a Convolutional Neural Network (CNN) such as ResNet or VGG.
Feature Extraction:
Extract features from the pre-trained model up to a certain layer, typically before the classification head.
Fine-tuning Phase:
Add a new classification head with categories corresponding to the unknown classes.
Fine-tune only the last few layers while freezing earlier layers.
Domain Adaptation Techniques:
Utilize techniques like Domain Adversarial Network Training (DAN) or Generative Adversarial Networks (GANs) to align feature distributions between source and target domains.

#### Implementation
Steps

Load Pre-trained Model:

PYTHON
import torch
from torchvision.models import resnet50

model = resnet50(pretrained=True)

Freeze Parameters:

Freeze the parameters of all layers except the new classification head.
PYTHON
for param in model.parameters():
    param.requires_grad = False

# Unfreeze the last few layers
for name, child in model.named_children():
    if 'layer4' not in name:
        for param in child.parameters():
            param.requires_grad = True

Add New Classification Head:

PYTHON
from torch import nn

num_features = model.fc.in_features
new_classes = 10  # Number of unknown classes

# Modify the last fully connected layer to output for new classes
model.fc = nn.Linear(num_features, new_classes)

model.to(device)  # Ensure it is on the appropriate device (CPU/GPU)

Train Model:

Use a smaller learning rate and less epochs compared to initial pre-training.
PYTHON
optimizer = torch.optim.SGD(model.parameters(), lr=0.001, momentum=0.9)

# Define loss function
criterion = nn.CrossEntropyLoss()

for epoch in range(num_epochs):
    running_loss = 0.0
    for inputs, labels in dataloader:
        optimizer.zero_grad()

        outputs = model(inputs)
        loss = criterion(outputs, labels)

        loss.backward()
        optimizer.step()

        running_loss += loss.item() * inputs.size(0)

print(f"Epoch {epoch} Loss: {running_loss / len(dataloader.dataset)}")

Domain Adaptation (Optional):

Implement techniques like DAN or use GANs to align feature distributions.
PYTHON
# Example of implementing Domain Adversarial Network Training (DAN)
from torchvision.models import vgg16_bn

def domain_adversarial_network_training(source_dataloader, target_dataloader):
    for data in source_dataloader:
        images, labels = data
        pred_classes, _ = model(images)

        # Calculate the cross-entropy loss on the prediction classes
        class_loss = criterion(pred_classes, labels)

        # Adversary loss: maximize the domain classifier's classification accuracy
        domain_classifier_output = domain_classifier(features)
        adversary_loss = adversarial_criterion(domain_classifier_output, torch.ones_like(domain_classifier_output))

        total_loss = (class_loss + alpha * adversary_loss) / 2

        optimizer.zero_grad()
        total_loss.backward()
        optimizer.step()

#### Closing Summary
and Common Mistakes/Follow-ups

By leveraging transfer learning and fine-tuning a pre-trained model, we can effectively classify unseen categories. However, it's crucial to balance the number of layers frozen versus unfrozen to avoid overfitting. Domain adaptation techniques help in aligning feature distributions between different domains but require careful tuning.

#### Common mistakes
include:

Overfitting on the new classes due to insufficient data.
Not properly balancing the loss functions during fine-tuning.
Ignoring data augmentation, which can significantly improve performance.

For follow-ups, consider discussing hyperparameter tuning strategies, evaluation metrics (e.g., F1 score, confusion matrix), and real-world application examples like medical image classification or autonomous driving.

**Comparison:**

| Layer | Type    | Parameters |
|-------|---------|------------|
| 0     | Conv2d  | 1024       |
| 1     | ReLU    |            |
| 2     | MaxPool |            |
| ...   | ...     |            |
| 67    | Linear  | 10         |

**Code:**

```python
import torch
from torchvision.models import resnet50

model = resnet50(pretrained=True)
for param in model.parameters():
    param.requires_grad = False

# Unfreeze the last few layers
for name, child in model.named_children():
    if 'layer4' not in name:
        for param in child.parameters():
            param.requires_grad = True

num_features = model.fc.in_features
new_classes = 10
model.fc = nn.Linear(num_features, new_classes)
optimizer = torch.optim.SGD(model.parameters(), lr=0.001, momentum=0.9)
criterion = nn.CrossEntropyLoss()
```

**Example input / output:**

```text
# Example input and output for fine-tuning phase
inputs = torch.randn(32, 3, 224, 224)  # Randomly generated inputs for demonstration
labels = torch.randint(low=0, high=new_classes, size=(32,))
optimizer.zero_grad()
outputs = model(inputs)
loss = criterion(outputs, labels)
print(f"Loss: {loss.item()}")
```

**Data:**

```json
{
    "source_images": [
        "image1.jpg",
        "image2.jpg"
    ],
    "target_images": [
        "target1.jpg",
        "target2.jpg"
    ]
}
```

**Flow:**

1. Load Pre-trained Model
2. Fine-tune Last Few Layers
3. Add New Classification Head
4. Train Model
5. Domain Adaptation (Optional)

