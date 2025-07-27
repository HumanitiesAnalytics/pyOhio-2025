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
import filecmp
import os
import subprocess

def are_dir_trees_equal(dir1, dir2):
    
    dirs_cmp = filecmp.dircmp(dir1, dir2)
    if len(dirs_cmp.left_only)>0 or len(dirs_cmp.right_only)>0 or \
        len(dirs_cmp.funny_files)>0:
        return False
    (_, mismatch, errors) =  filecmp.cmpfiles(
        dir1, dir2, dirs_cmp.common_files, shallow=False)
    if len(mismatch)>0 or len(errors)>0:
        return False
    for common_dir in dirs_cmp.common_dirs:
        new_dir1 = os.path.join(dir1, common_dir)
        new_dir2 = os.path.join(dir2, common_dir)
        if not are_dir_trees_equal(new_dir1, new_dir2):
            return False
    return True

are_dir_trees_equal('test_source', 'test_target')
```

```python
command = f"rsync -avun --delete 'test_source/' 'test_target'"
result = subprocess.run([command], shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)  
```

```python
try:
    print(result.stdout.decode())
except UnicodeDecodeError:
    print(result.stdout.decode('latin-1'))
```

```python

```
