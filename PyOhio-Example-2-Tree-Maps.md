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
import os
import pandas as pd 

def fs_tree(root, mc=200):
    results = {}
    results['file_count_max'] = mc
    results['root'] = root
    for (dirpath, dirnames, filenames) in os.walk(root):
        parts = dirpath.split(os.sep)
        if len([i for i in parts if i.startswith('.')]) < 1:
            curr = results
            
            for p in parts:
                if p != '':
                    curr = curr.setdefault(p, {})
                    
            curr['this_folder'] = parts[-1]
            curr['disk_usage'] = assess_disk_usage(dirpath)
            curr['disk_percent'] = curr['disk_usage']/results[root]['disk_usage']
                
            curr['file_count'] = file_count(dirpath, mc)
            curr['depth'] = len([ i for i in parts if parts !='']) -1
            
            if curr['depth'] < 1:
                curr['parent'] = None
            else:
                
                curr['parent'] = parts[curr['depth']-1]
    return results

# skip hidden folders 

def assess_disk_usage(location):
    
    total_size_bytes = 0
    
    total_size_bytes += os.path.getsize(location)
    
    for root, dirs, files in os.walk(location, topdown=True):
        
        # walk folders 
        for name in dirs:
            if not name.startswith('.'):
                this_folder = os.path.join(root, name)
                total_size_bytes += os.path.getsize(this_folder)
    
    # gb = divide by 10_000
    return total_size_bytes

def file_count(location, max_count=200):
    c = 0
    for root, dirs, files in os.walk(location, topdown=True):
        for f in files:
            full = os.path.join(root, f)            
            if os.path.isfile(full) and not f.startswith('.'):
                c+=1
            if c == max_count:
                return c
        return c

def flatten_tree_recursive(tree_data, rows =[], meta = ['this_folder', 'disk_usage', 'disk_percent', 'file_count', 'depth', 'parent']):
    r = rows
    for i in tree_data.keys():
        if i not in meta:
                       
            this_dict = tree_data[i]
            if type(this_dict) == dict:
                row = []
                for k in meta:
                    row.append(this_dict[k])
                r.append(row)
                r = flatten_tree_recursive(this_dict, rows=r)
    return r
```

```python
src_result = fs_tree('test_source')
src_flat = flatten_tree_recursive(src_result, rows=[])
meta = ['this_folder', 'disk_usage', 'disk_percent', 'file_count', 'depth', 'parent']
df_src_flat = pd.DataFrame(src_flat, columns=meta)
df_src_flat
```

```python
trg_result = fs_tree('test_target')
trg_flat = flatten_tree_recursive(trg_result, rows=[])
df_trg_flat = pd.DataFrame(trg_flat, columns=meta)
df_trg_flat
```

```python
df_src_flat['source'] = 1
df_trg_flat['target'] = 1
df_joined = df_src_flat.set_index(['this_folder', 'parent', 'depth']).join(df_trg_flat.set_index(['this_folder', 'parent', 'depth']), lsuffix='_source', rsuffix='_target', how='outer').reset_index().fillna(0)
df_joined['total'] = df_joined['source'] + df_joined['target']
df_joined.loc[df_joined['total'] != 2][['this_folder', 'parent', 'depth', 'source', 'target', 'total']]
```

```python
df_joined.loc[df_joined['total'] == 2][['this_folder', 'parent', 'depth', 'source', 'target', 'total']]
```

```python
try:
    assert all(df_joined['total'] == 2.0)
    print("True")
except Exception as e:
    print(df_joined['total'].tolist())
```

```python

```
