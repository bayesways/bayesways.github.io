---
layout: post
date: 2025-05-10
title: How I set up my project repositories
giscus_comments: true
related_posts: false
tags: 
 - productivity
---
### My Simple Python Project Setup

After trying different tools and structures, this is the setup I now use for each project:

---

### 0. Check Python 3

Run in Terminal:

```bash
python3 --version
```

If you see a version 3 you are good. Otherwise you might want to install Python3 via [Homebrew](https://brew.sh).

---

### 1. Install Pyenv
Follow instructions [here](https://github.com/pyenv/pyenv)

```bash
brew install pyenv
```

If you are on zsh do the following: 

```bash
  echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
  echo '[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
  echo 'eval "$(pyenv init - zsh)"' >> ~/.zshrc
```

otherwise follow the instructions here to [setup your terminal](https://github.com/pyenv/pyenv?tab=readme-ov-file#b-set-up-your-shell-environment-for-pyenv). 

---

### 2. Create a Project Folder

```bash
cd ~/my_repos/
mkdir project_name
cd project_name
```

---

### 3. Use `pyenv` to Set Python Version

```bash
pyenv install -l         # view available versions
pyenv install 3.12.0     # install version
pyenv local 3.12.0       # set for project
python --version         # confirm
```

This will create a `.python-version` file in the repo. You would want to commit this to git if you plan to install the repo on a new computer later. 

---

### 4. Create & Activate a Virtual Environment

```bash
python -m venv venv
source venv/bin/activate 
```

This will create a `venv` folder in your project folder which is specific to your system. You don't have to commit this to your git but you can. When you activate the environment in your terminal you should see `(venv)` in your terminal prompt. To get out of the environment run `deactivate`. Note that it's good practice to always name your virtual environment `venv` as it is a unique folder that always lives inside a project folder with a unique name. 

---

### 5. Upgrade Pip & Install Packages

```bash
pip install --upgrade pip
pip install pandas llm  # list any other package as needed
pip freeze > requirements.txt
```

This will create a `requirements.txt` file in your project. You want to commit this to your git repo if you plan to install the repo later on a new computer later. If you want to install another package later you repeat the same steps

```bash
source venv/bin/activate # make sure you are working from inside the virtual environment
pip install llm-mxl # install new package 
pip freeze > requirements.txt # update requirements file
```


---

### 6. Fresh install on a new computer
On a new computer you can clone the repo, create a virtual environment, install the python version dictated in `.python-version` and install all the requirements as follows
```bash
# 1. Clone the repo (replace with your repo URL)
git clone https://github.com/your-username/your-project.git
cd your-project

# 2. Install the Python version from .python-version via pyenv
pyenv install $(cat .python-version)

# 3. Set the local Python version for the project
pyenv local $(cat .python-version)

# 4. Create a virtual environment named 'venv'
python -m venv venv

# 5. Activate the virtual environment
source venv/bin/activate

# 6. Upgrade pip
pip install --upgrade pip

# 7. Install dependencies from requirements.txt
pip install -r requirements.txt

```


That’s it—a clean, repeatable Python project setup.
