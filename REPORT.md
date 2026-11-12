
1. Baseline
The goal of this task was to improve the leaf classifier from Task 3 using a coding agent.
The original CNN model was used as the baseline. It reached 66.7% validation accuracy and 41.4% camera-like accuracy.

Two datasets were used: 
    - Our dataset: 107 training images and 29 validation images collected by our group during the lab.
    - Historical dataset: 700 training images and 120 validation images from previous years, provided through the link in the assignment.

The main experiments were trained only on our 107 images. The same training images and fixed random seeds were used for a fair comparison.
A camera-like validation set was also created from our 29 validation images by applying random crops, rotations and colour changes. The same modified images were used to evaluate every model.

The historical validation images were used to check how well the models worked on photos taken by other students. In separate experiments, the 700 historical training images were added to our training data, giving 807 training images in total.

2. Experiments and Results
Different improvements were tested, first separately and then in combinations. The same training images and fixed random seeds were used for a fair comparison.
Camera-like accuracy was the main focus because the classifier should work with phone photos.

| Experiment | Camera accuracy | Change |
|---|---:|---:|
| Baseline CNN | 41.4% | — |
| Geometric augmentation | 50.6% | +9.2 pp |
| ColorJitter + augmentation | 54.0% | +12.6 pp |
| Dropout | 43.7% | +2.3 pp |
| Global average pooling* | 48.3% | +6.9 pp |
| AdamW* | 37.9% | −3.5 pp |
| Label smoothing* | 34.5% | −6.9 pp |
| Best epoch weights | 44.8% | +3.5 pp |
| Frozen pretrained model | 63.2% | +21.8 pp |
| Fine-tuned pretrained model | 58.6% | +17.2 pp |
| Pretrained + augmentation + cosine | 67.8% | +26.4 pp |
| Fine-tuned + jitter + cosine | 70.1% | +28.7 pp |
| Frozen + jitter + cosine | 72.4% | +31.0 pp |

*These experiments used one seed. Other results are averages of three seeds. Changes are compared with the baseline.*
The biggest improvements were achieved with transfer learning and augmentation. ColorJitter also improved the result compared with geometric augmentation alone.

3. Noise and What Did Not Help
The validation set contained only 29 images, so the results were sensitive to random variation.
The baseline camera-like accuracy had a standard deviation of 4.9 percentage points across three seeds.
Small differences, such as dropout (+2.3 pp) and keeping the best epoch (+3.5 pp), were within this variation. AdamW and label smoothing did not improve the results. Global average pooling reduced the model size, but its improvement was not reliable.
Transfer learning improved camera-like accuracy by 17–31 percentage points, which was much larger than the observed variation.

4. Additional Training Data

An additional experiment was performed using 700 historical training images from previous years.
The fine-tuned model with extra data reached 90.8% accuracy on the historical validation set and 72.4% on the camera-like set.
These results suggest that more training images can help the model recognize leaves from different sources.

5. Final Model
For the phone application I would select MobileNetV3-small with a frozen backbone, ColorJitter and a cosine learning rate.
It reached 72.4% camera-like accuracy and 78.2% validation accuracy on average. The exported ONNX model is about 6.4 MB.
However, the fine-tuned model with extra historical data performed better on historical photos, so it could be a better choice when more training data are available.

6. Limitations
The camera-like validation set was created by modifying existing images, not by taking real phone photos.
The dataset was also small, so the results may be different on new images.

The selected ONNX model was also tested in the camera application. The model loaded successfully and was able to classify leaf images.