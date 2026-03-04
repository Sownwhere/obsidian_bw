


# Question 1
```bash
ImportError: libpython3.8.so.1.0: cannot open shared object file: No such file or directory
```






#### 🔄 2. **Link the library manually (not ideal, but works)**

If the `.so` file exists somewhere on your system (e.g., `/usr/local/lib` or `~/.pyenv`), you can export the path:


```bash
export LD_LIBRARY_PATH=/path/to/lib:$LD_LIBRARY_PATH
```

Replace `/path/to/lib` with the actual path where `libpython3.8.so.1.0` is located.

You can locate it with:

```bash 
find / -name "libpython3.8.so.1.0" 2>/dev/null
```


---


```bash
	export LD_LIBRARY_PATH=//home/lawrance/miniconda3/envs/omnih2o/lib:$LD_LIBRARY_PATH

export LD_LIBRARY_PATH=/home/lawrance/anaconda3/pkgs/python-3.8.20-he870216_0/lib:$LD_LIBRARY_PATH


	export LD_LIBRARY_PATH=/home/bowen/anaconda3/pkgs/python-3.8.20-he870216_0/lib:$LD_LIBRARY_PATH

```    





# Question 2

```bash
/home/lawrance/anaconda3/envs/isaacgymenvs/bin/../lib/libstdc++.so.6: version `GLIBCXX_3.4.32' not found (required by /home/lawrance/.cache/torch_extensions/py38_cu117/gymtorch/gymtorch.so)`
```


### ✅ Solution

#### Option 1: **Use System libstdc++ Instead of Conda’s**

Override the Conda version of `libstdc++` with your system's (usually newer):


```bash
export LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libstdc++.so.6
```


Then try running your script or importing the module again.

> 💡 Check where your system `libstdc++.so.6` is located:


```bash

strings /usr/lib/x86_64-linux-gnu/libstdc++.so.6 | grep GLIBCXX

```


You should see `GLIBCXX_3.4.32` in the output.



[2.5920, 2.5920, 3.1416, 3.1416, 4.9637, 4.9637, 0.9090, 0.9090, 1.2566,
        1.4154, 0.4712, 0.4712]