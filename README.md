# Pneumonia-triage-assistant
Track: Health
Team: names
> **This project is a prototype and is not medical advice.**
The problem
In many small and rural clinics there is no radiologist on site, so chest X-rays wait hours or days to be read.
A child with pneumonia can sit in the same queue as healthy patients. Our user is a nurse or general doctor at
such a clinic who needs to know which X-rays to send for reading first.
What we built
A chest X-ray image goes in. The tool returns a pneumonia probability and a triage label:
🔴 URGENT (read first), 🟡 UNSURE (a person must review it soon) or 🟢 LIKELY NORMAL (normal queue).
Staff upload the scan in a simple web app (Gradio) and reorder the reading queue based on the label. The app
shows a coloured result card and a short plain-language explanation written by Gemini from the model's numbers.
Every scan is still read by a clinician. The tool only changes the order.
What we added
We started from suggested project 4.1.1 (Pneumonia screening from chest X-rays) and added:
Our own setting: triage for a clinic with no radiologist, instead of plain normal/pneumonia classification.
Handling unsure scans: a middle "UNSURE" band so the model isn't forced into a yes/no answer. Its limits
are chosen on validation data so that ≥99% of validation pneumonia cases score above the lower edge.
A threshold chosen for safety: we treat a missed pneumonia as worse than a false alarm and pick the threshold
that catches ≥95% of validation pneumonia cases, instead of the default 0.5.
How it works
Data: PneumoniaMNIST (MedMNIST v2), 64×64 grey chest X-rays with the official splits: 4,708 train, 524
validation and 624 test images. About 74% of the training images show pneumonia. Pixels are scaled to 0–1. We
add small random zoom, shift and brightness changes during training only.
Models:
Baseline: always predict "pneumonia".
Small CNN (3 convolution blocks) trained from scratch, with a class-weighted loss for the imbalance.
ResNet-18 pretrained on ImageNet and fine-tuned (images copied to 3 channels and enlarged to 128 px).
We kept the best epoch and model by validation AUC. The test set was only used for the final numbers.
Results
Paste the summary table from the last notebook cell here.
Model	Accuracy	Recall (pneumonia caught)	Missed cases (FN)	False alarms (FP)	AUC
Baseline: always pneumonia	0.625	1	0 	234 	–
Small CNN (0.5)	0.947	0.992	3	30 	0.989
ResNet-18 (0.5)	0.958	0.992	3	23 	0.994
Chosen model (safety threshold)	0.96	0.979	8	23 	0.994 
Triage on the test set: X URGENT, Y UNSURE, Z LIKELY NORMAL. N pneumonia cases ended up in the
LIKELY NORMAL pile.
Where it fails: Describe 1–2 missed cases and 1–2 false alarms from section 7 of the notebook, and why you
think they happen.
Limits: all images are X-rays of children aged 1–5 from a single hospital in Guangzhou. The dataset has no
age, sex or scanner information, so we could not check whether the model works equally well across groups. We
expect it to perform worse on adults and on images from other hospitals.
What we'd do next
Test the model on chest X-rays from a different hospital or country, and on adults.
Show clinic staff the app, ask whether the three labels match how they would actually work, and adjust the
thresholds to fit their workload.
Sources
MedMNIST v2 / PneumoniaMNIST: Yang et al., 2023, Scientific Data. https://medmnist.com
Original images: Kermany et al., 2018, Cell, "Identifying Medical Diagnoses and Treatable Diseases by
Image-Based Deep Learning". https://data.mendeley.com/datasets/rscbjbr9sj/2
PyTorch and torchvision (ResNet-18 ImageNet weights): https://pytorch.org
scikit-learn: https://scikit-learn.org · Gradio: https://gradio.app
Google Colab: https://colab.research.google.com
Gemini API (Google, free tier), used in the demo app only to explain the model's result in plain language. It
receives only the numbers, never the X-ray. https://ai.google.dev
AI coding assistant: Claude (Anthropic) helped write the notebook code
AI Support: Gemini(Google) provided answers and troubleshooted the code
Judges: Please Review at https://youtu.be/fH91hx7HEpw
Team Nexus: Jasper, Girikshith, Vihaan
