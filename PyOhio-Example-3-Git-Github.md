---
jupyter:
  jupytext:
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.3'
      jupytext_version: 1.16.6
  kernelspec:
    display_name: Python 3 (ipykernel)
    language: python
    name: python3
---

```python
import subprocess
import os
import time
```

```python
def get_repos(root):
    '''traverse contents of a target folder and return top-level folders 
       containing git repositories ... returns a list of folder names'''
    
    if root.startswith('/'):
        root_folder = root.split('/')[-1]
        preamble = root.replace('/'+root_folder, '/')
    else:
        root_folder = root
        preamble = ''
    gits = []
    for (dirpath, dirnames, filenames) in os.walk(root):
        if '.git' in dirnames and 'ignore/' not in dirpath:
            gits.append(dirpath)
    return gits

def get_status(repo_path):
    '''passes 'git status' command via subprocess to assess current repo state
       ... returns result of subprocess.run '''
    
    command = f"""cd {repo_path}
    git status
    cd"""
    r = subprocess.run([command], shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
    return r

def pull_push(repo_path, msg='adding updated files'):
    '''passes git commands via subprocess to pull from origin, add, commit, and 
       push to origin ... returns result of subprocess.run '''
    
    command = f"""cd {repo_path}
    git pull origin main
    git add . 
    git commit -m '{msg}'
    git push origin main
    cd"""
    r = subprocess.run([command], shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)  
    return r
    
```

```python
# modify this path to locate a folder on your system containing git repos 
root_folder = '../../teaching'
repos = get_repos(root_folder)
len(repos) 
```

```python
repos[0]
```

```python
response = get_status(repos[0])

try:
    print(response.stdout.decode())
except UnicodeDecodeError:
    print(response.stdout.decode('latin-1'))
```

```python
response = pull_push(repos[0])

try:
    print(response.stdout.decode())
except UnicodeDecodeError:
    print(response.stdout.decode('latin-1'))
```

```python

```
