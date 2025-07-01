---
layout:     post
title:      "Data Leakage in Data Science" 
subtitle:   "The model cheats by seeing unwanted information"
author: "Dan Guitron"
catalog: true
header-img: "img/data-leakage.png"
header-mask: 0.4
tags:
  - Data Science
  - Data Leakage
  - ML
  - Machine Learning
  - Model Training
---



# Data Leakage in Data Science
***
>[!cue] What is data leakage?

`Data Leakage` in DS occurs when information from the test (or validation) set inadvertently "leaks" into the training process. It essentially allows your model to **"cheat"**  by seeing information it would't have access to in a real-world scenario.

>[!Note] Causing your model to perform unrealistically well during development but poorly on truly unseen data in the real world. 

>[!Important] The Critical Rule: Split BEFORE Preprocessing! ⚠️

## 📓Notes
---
It' is absolutely crucial in data science to **split your data into training and test sets before performing any imputation**, or indeed almost any other data preprocessing steps that involve learning parameters from the data (like scaling/normalization). The primary reason for this is to prevent data leakage. 


## 📝Examples
---

### Estimating Housing Prices with Missing Data

- 📝**Problem:** Imagine that we will developing a model to predict the price for a housing `housing_price` based in some features. One of this is `num_rooms` with some `NaN` values.
- 🎯**Target:** Predict the price of one house `housing_price`

**Dataset**

| id_housing | square_meters | num_rooms | years_old | housing_price |
| :--------- | :------------ | :-------- | :-------- | :------------ |
| 1          | 150           | 3         | 10        | 200,000       |
| 2          | 200           | 4         | 5         | 280,000       |
| 3          | 120           | **NaN**   | 15        | 160,000       |
| 4          | 180           | 3         | 8         | 240,000       |
| 5          | 250           | **NaN**   | 3         | 350,000       |
| 6          | 170           | 3         | 12        | 220,000       |
| 7          | 210           | 4         | 6         | 300,000       |
| 8          | 130           | **NaN**   | 20        | 175,000       |
| 9          | 230           | 4         | 4         | 320,000       |
| 10         | 190           | 3         | 9         | 260,000       |

>[!Note] We have `NaN` values in `num_rooms`


#### ❌ Case 1: Error! Imputation Before Splitting - (Data Leakage)

1.**Calculating the global mean:**
	- First, we calculate the mean of `num_rooms` using all the data in the column (including those of the test set)
	- known values: $3,4,3,3,4,3$.
	- Global mean: $(3+4+3+3+4+3)/6=20/6≈3.33$

2.**Imputing all `NaN`:**
	- Now, we replace all the `NaN` in `num_rooms` with 3.33.

| id_housing | square_meters | num_rooms | years_old | housing_price |
| :--------- | :------------ | :-------- | :-------- | :------------ |
| 1          | 150           | 3         | 10        | 200,000       |
| 2          | 200           | 4         | 5         | 280,000       |
| 3          | 120           | **3.33**  | 15        | 160,000       |
| 4          | 180           | 3         | 8         | 240,000       |
| 5          | 250           | **3.33**  | 3         | 350,000       |
| 6          | 170           | 3         | 12        | 220,000       |
| 7          | 210           | 4         | 6         | 300,000       |
| 8          | 130           | **3.33**  | 20        | 175,000       |
| 9          | 230           | 4         | 4         | 320,000       |
| 10         | 190           | 3         | 9         | 260,000       |

3. **Splitting the data (now with leaks):**
	- Split the dataset (now imputed) in train and test datasets.
	- **Train dataset (e.g: `id_housing` 1,2,3,4,5,6,7,9):**
		- Contains value of `num_rooms` that were imputed using the mean that included the actual values of the rows that will end up in the test dataset. 
		- E.g: row 5 (`num_rooms = 3.33`) was imputed using the mean that **knew** the value of the row 10 (which is 3 and goes to the test dataset.)
	- **Test dataset (e.g: `id_housing` 8,10):**
		- row 8 (`num_rooms = 3.33`)  was also imputed using the mean that considered the actual values of the training dataset rows.

4. **Training & Evaluation of the Mode:l**
	- The model is trained with the **"contaminated"** training dataset.  
	- When evaluated on the test dataset, the model appears to perform excellently. Why? Because the imputation on the test dataset are already subtly **"biased"** by the information the model indirectly **"learned"** from the training dataset during imputation. There is an artificial correlation. 

>[!Note] Consequence: The model looks better than it really is. When you deploy the model to the real world with new and unknown data, model performance will be much worse because it will not have that indirect **"help"** from future values.

#### ✅Case 2: Correct! Imputation After Splitting - (Without Leakage)

1. **Splitting the data (First!):**
	- Split the original dataset (with the intact `NaN`s ) in training and test dataset.
	- **Training dataset: (e.g `id_housing` 1,2,3,4,5,6,7,9)**
		- `num_rooms` values: 3,4,NaN,3,NaN,3,4,4
		- **This will be the ONLY dataset used to learn the imputation rules!**
	- **Test dataset: (e.g `id_housing` 8,10)** 
		- `num_rooms` values: NaN,3
2. **Mean training dataset calculation:**
	- Calculate the mean of `num_rooms` **ONLY** using the data of the training dataset. 
	- Known values of training: 3,4,3,3,4,4.
	- Mean of training (3+4+3+3+4+4)/6=21/6=3.5
3. **Imputing for both datasets (using the mean of training):**
	- Now, we use the mean of training (3.5) to impute the `NaN` of both datasets.
	- **Training dataset (imputed):** 
		- `num_rooms`: (3,4,3.5,3,4,4)
	- **Test dataset (imputed with the same mean):**
		- `num_rooms`: 3.5, 3
4. **Training & Evaluation of the Model:**
	- The model is trained with the training dataset (correctly imputed).
	- When evaluated on the test dataset, the performance will have a more **realistic** estimation of how will behave with new and unknown data. The imputations on the test dataset were based only on  information the model may have **"learned"** from training, without **"seeing"** into the future. 

>[!Note] Consequence: You get an honest assessment of you model's generalization ability. If your model perfoms well with this approach, it's much more likely to perform well in the real world. 
