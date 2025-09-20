---
layout: post
title: 🐍 Use Anaconda to Create a Clean Python Environment 
categories: [Python,]
excerpt: How to Use Anaconda to Create a Clean Python Environment
image: 
hidden: True 
---

Setting up a clean and stable Python environment is essential for learning and working on projects. This guide will show you how to use **Anaconda** to do just that.

## 📦 What is Anaconda?

Anaconda is a free Python distribution that includes:

- Python interpreter
- Popular libraries like NumPy, pandas, matplotlib
- Jupyter Notebook
- `conda` environment manager

It's ideal for students and beginners because it simplifies installation and avoids conflicts between projects. It is compatible with [VS CODE](https://code.visualstudio.com/) ide. An official tutorial exist [here](https://test-jupyter.readthedocs.io/en/latest/install.html).

## 🔧 Step 1: Install Anaconda

1. Download Anaconda:  
   👉 [https://www.anaconda.com/download](https://www.anaconda.com/download)

2. Choose your OS and download the **Python 3.x (64-bit)** version.

3. Run the installer and follow the prompts:
   - (Optional) On Windows, check "Add Anaconda to my PATH"
   - Select "Just Me" and complete installation.

## 🧪 Step 2: Create a New Conda Environment

Open **Anaconda Prompt** (Windows) or Terminal (macOS/Linux):
```bash
conda create -n myproject python=3.10
```

## 📂 Step 3: Activate Your Environment

After creating the environment, you need to activate it:
```bash
conda activate myproject
```

## 📚 Step 4: Install Packages

Use conda install to install packages:

```bash
conda install numpy pandas matplotlib jupyter
```
If a package isn’t available in the default conda channel, you can try conda-forge:

```bash
conda install -c conda-forge scikit-learn
```
Or even use pip inside the environment:

```bash
pip install seaborn
```
Now any Python packages you install will be isolated from your base installation.

## 💻 Step 5: Use the Environment in VS Code

Anaconda integrates smoothly with VS Code. Once your environment is created, you can use it for both Jupyter notebooks and Python scripts.

### 📝 For Notebooks (.ipynb)

1. Open VS Code.
2. Install the extensions:
3. Python (by Microsoft)
4. Jupyter (by Microsoft)
5. Open a .ipynb file or create a new notebook.
6. In the top-right corner, select the kernel → choose your conda environment (myproject).

Now, your notebook cells will run inside the clean conda environment.

### 📄 For Python Scripts (.py)

1. Open a .py file in VS Code.
2. Look at the bottom-right corner of VS Code → click on the interpreter selector.
3. Choose the interpreter from your environment (it should appear as something like Python 3.10 ('myproject')).

Once selected, running scripts (Run → Run Without Debugging or the green button) will execute in your conda environment.