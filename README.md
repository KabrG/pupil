# Pupil: A Modest Image Captioning Model

**Author:** Kabir Guron<br>
**Contact:** kabirguron@gmail.com<br>
**Notebook:** [Google Colab Notebook](https://colab.research.google.com/drive/14K0RnUWMn5XxLJ2dT5a6LbogBHtTDbEO)

## Introduction
Pupil is a Vision Transformer (ViT) and Transformer decoder model built for image captioning. It combines spatial representation learning from a pre-trained ViT encoder with sequence generation from a decoder trained from scratch. Transforming visual data into natural language has practical applications in e-commerce, dataset curation, and autonomous systems.

## Architecture
The system pairs a frozen Vision Transformer encoder with a 6-layer Transformer decoder:
* **Encoder:** Pre-trained ViT model operating on $16 \times 16$ image patch embeddings. The original classification head was removed to yield a 768-dimensional latent representation. Encoder weights remain frozen during training.
* **Decoder:** 6-layer Transformer built from scratch with 12 attention heads, 768 token embedding dimensions, and a maximum sequence length of 64 tokens. Includes $10\%$ dropout for regularization.
* **Inference Strategy:** Uses beam search (beam width = 10) with low temperature sampling for sequence generation.
* **Model Scale:** 220M total parameters (133M trainable).

<p align="center">
  <img width="700" alt="pupil_diagram" src="https://github.com/user-attachments/assets/9128613c-1972-4bbf-a496-7e36ac8d1433" />
</p>

## Data Processing
* **Dataset:** COCO 2017 dataset (120K train images with 5 captions each, total 590K training pairs; 5K validation images; 40K test images).
* **Image Pipeline:** Resized to $224 \times 224$ pixels. Normalization using standard ImageNet mean `[0.485, 0.456, 0.406]` and std `[0.229, 0.224, 0.225]`. Training set augmentations include random shifts in brightness, contrast, and saturation.
* **Text Pipeline:** Tokenization via GPT-2 tokenizer with `<BOS>`, `<EOS>`, and padding tokens capped at 64 tokens per caption.

<p align="center">
  <img width="480" alt="transformation_compare-2" src="https://github.com/user-attachments/assets/4dd5c8d8-643d-440e-bfe0-06db5bc5f875" />
</p>

## Baseline Comparison
To evaluate spatial encoding performance, a baseline model replaced the ViT encoder with a fully connected Artificial Neural Network (ANN) that flattens the input image into a 2-layer MLP with a 2048-unit hidden layer and ReLU activation. The baseline used the exact same decoder setup.

## Results

### Quantitative Metrics
* **Loss:** Pupil achieved a final validation loss of 2.54, compared to the baseline loss which stalled around 11.2.
* **BLEU:** Pupil recorded an average BLEU score of $9.52\%$ across 1,000 random validation samples (baseline score: $0.01\%$).
* **Human Rating (COCO Test):** Evaluated by a blind 3rd-party reviewer on a 1-4 scale (1 = unrelated, 4 = accurate and complete). Pupil achieved an average score of 3.48 across 100 test images.

<p align="center">
  <img width="500" alt="bleu_score" src="https://github.com/user-attachments/assets/e31fcd00-501d-47ff-b2a8-2508df745576" />
</p>

### Qualitative Performance
* Pupil successfully identifies age groups, object positions, colors, and spatial relationships (e.g., "a person sitting on a couch using a laptop").
* The baseline model failed to output coherent or relevant sentences across 50 observed test runs.

<p align="center">
  <img width="720" alt="pred_sample" src="https://github.com/user-attachments/assets/8a79c732-6976-4a1d-9faf-fd330c65fe1a" />
</p>

### Performance on Unseen Real-World Data
Tested on 70 personal images provided by 5 individuals without hyperparameter adjustments:
* **High Detail (Score 4):** 19 samples
* **Minor Errors (Score 3):** 23 samples
* **Remotely Related (Score 2):** 15 samples
* **Unacceptable/Inaccurate (Score 1):** 13 samples
* **Average Human Score:** 2.69 / 4.00

<p align="center">
  <img width="720" alt="good_predictions" src="https://github.com/user-attachments/assets/e7c35307-84fd-4108-8f11-8a85e9cea767" />
</p>

## Failure Modes
1. **Numerical Specificity:** Difficulty accurately counting repeated objects in a scene.
2. **Out-of-Distribution Objects:** Reduced accuracy on objects rare or absent in the COCO dataset.
3. **Class Representation Bias:** Tendency to predict overrepresented classes (e.g., mistaking a wristwatch for a clock due to the prevalence of clock towers in training data).

<p align="center">
  <img width="720" alt="poor_predictions" src="https://github.com/user-attachments/assets/c9dfa736-cae2-4d5c-8cab-4bcf740c0659" />
</p>

## Acknowledgements
* Semester solo project for APS360 at the University of Toronto.
* The authors of *An Image is Worth 16x16 Words: Transformers for Image Recognition at Scale* for the pre-trained Vision Transformer concepts and weights.
* The creators of the COCO 2017 dataset and OpenAI for the open-source GPT-2 tokenizer.
* My friends who provided testing data for my model to perform inference on.
