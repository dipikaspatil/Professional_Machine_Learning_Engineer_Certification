# AI Models
## 1. Terminology Hierarchy & Definitions

* **Artificial Intelligence (AI):** Broad umbrella term encompassing any system capable of mimicking human intelligence (e.g., robotics, autonomous vehicles).
* **Machine Learning (ML):** Subset of AI focusing on systems that discover patterns and learn from data without explicit procedural programming.
* **Deep Learning (DL):** Subset of ML that uses multi-layered neural networks between input and output layers to capture complex patterns.
* **Generative AI (GenAI):** Subset of DL that utilizes foundation models (e.g., Large Language Models) to generate content, interpret data, and perform interactive tasks.

![Screenshot 2026-08-31 at 11.09.05 AM.png](../images/1.png)


![Screenshot 2026-08-31 at 11.09.38 AM.png](../images/2.png)
---

## 2. Supervised vs. Unsupervised Learning

| Dimension | Supervised Learning | Unsupervised Learning |
| :--- | :--- | :--- |
| **Data Input** | Labeled data (Features + Target labels) | Unlabeled data (Features only) |
| **Approach** | Task-driven (Learns mappings to known targets) | Data-driven (Finds hidden patterns & structures) |
| **Primary Goal** | Predict outcomes for unseen inputs | Discover natural groupings or relationships |

---

## 3. Key Tasks and Algorithms

### Supervised Learning
* **Classification:** Predicts discrete categorical variables (e.g., Cat vs. Dog, Spam vs. Not Spam).
  * *Example Model:* **Logistic Regression**
* **Regression:** Predicts continuous numerical values (e.g., Forecasting product sales or customer spending).
  * *Example Model:* **Linear Regression**

![Screenshot 2026-08-31 at 11.12.19 AM.png](../images/3.png)
### Unsupervised Learning
* **Clustering:** Groups data points with similar characteristics together (e.g., Customer segmentation).
  * *Example Model:* **K-Means Clustering**
* **Association:** Identifies hidden relationships/correlations between variables (e.g., Market basket analysis).
  * *Example Model/Algorithm:* **Apriori Algorithm**
* **Dimensionality Reduction:** Compresses feature space while retaining key variance (e.g., Feature aggregation for risk assessment).
  * *Example Technique:* **Principal Component Analysis (PCA)**

![Screenshot 2026-08-31 at 11.13.21 AM.png](../images/4.png)
---

## 4. Practical Scenarios & Self-Assessment

### Scenario 1: Customer Spending Prediction
* **Problem:** Predict future customer spending based on historical purchase data.
* **Paradigms:** Supervised Learning (Data has explicit purchase amounts as labels).
* **Task Type:** Regression (Target variable is continuous).
* **Model Choice:** Linear Regression.

### Scenario 2: Customer Segmentation
* **Problem:** Group customers into natural segments without pre-assigned demographic labels.
* **Paradigms:** Unsupervised Learning (No pre-defined group labels).
* **Task Type:** Clustering (Grouping by feature similarity).
* **Model Choice:** K-Means Clustering.

---

## 5. Google Cloud Implementation Mapping
These foundational algorithms map directly to GCP model options:
* **BigQuery ML:** Train tabular models (Linear/Logistic Regression, K-Means) directly via standard SQL queries.
* **AutoML (Vertex AI):** Automate feature engineering and architecture selection for structured and unstructured datasets.
* **Custom Training (Vertex AI):** Build custom deep learning models using framework containers (TensorFlow, PyTorch, JAX).

## Additional info 

### Introduction to Supervised Learning

Think of supervised learning like a **teacher helping a student learn using flashcards**. Every flashcard has a question (input data) and the correct answer written on the back (the label). 

The goal is for the student to look at enough flashcards to figure out the underlying pattern so they can answer new, unseen questions correctly. Depending on what kind of answer is written on the back of those flashcards, the student is doing either **Classification** or **Regression**.

---

### 1. Classification: The "Grouping" Game

In classification, the answers on the flashcards are **labels, categories, or names**. Your goal is to put things into separate, distinct buckets. There is no middle ground or "in-between" values.

* **The Core Question:** "Which specific bucket does this belong to?"
* **Visualizing It:** Imagine a table covered in red apples and green limes. Classification is drawing a straight line down the middle of the table to perfectly separate the apples from the limes. If a new fruit arrives, you look at which side of the line it falls on.

### Simple Everyday Examples:
* **Email Filters:** Looking at an incoming email and throwing it into the "Spam" bucket or the "Inbox" bucket.
* **Medical Screening:** Looking at an X-ray and deciding if it is "Healthy" or "Unhealthy".
* **Animal App:** Snapping a picture of a pet and deciding if it is a "Dog", "Cat", or "Bird".

---

### 2. Regression: The "Guessing the Number" Game

In regression, the answers on the flashcards are **continuous numbers**. These are things you can measure, count, or calculate on a smooth scale. The answers can have decimals, can go up or down infinitely, and the difference between numbers matters mathematically.

* **The Core Question:** "How much?" or "How many?"
* **Visualizing It:** Imagine a graph showing house sizes and their prices. Regression is drawing a smooth line that passes right through the middle of all those scattered data points. If a new house size is introduced, you follow the line to see exactly what dollar amount it aligns with.

### Simple Everyday Examples:
* **Weather Forecasting:** Predicting that tomorrow's temperature will be exactly 72.5°F.
* **Real Estate:** Predicting that a house will sell for $420,000 based on its square footage.
* **Streaming Apps:** Predicting exactly how many minutes a user will watch a video before clicking away.

---

### Summary Shortcut

If you ever get confused, look at the type of output you want to predict:

* Is the answer a **word, a choice, or a category**? $\rightarrow$ **Classification**
* Is the answer a **measurable value, a price, or a number**? $\rightarrow$ **Regression**
