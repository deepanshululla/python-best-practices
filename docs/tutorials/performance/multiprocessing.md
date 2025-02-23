!!! Summary

    :white_check_mark: Partition data into independent batches/chunks.
    
    :white_check_mark: Use shared memory for large data.
 
    :x: Avoid naive element-wise processing.

    :x: Avoid Using multithreading for CPU-bound tasks.

    :x: Avoid Using multiprocessing for IO-bound tasks.


# Multi-Processing

Due to the infamous python GIL (Python 3.13 have flags to disable it in experiment), when you need more CPU power to crunch some data, multi-processing is the way to go.   Python has a built-in multiprocessing library that make this feature avaiable out of the box. 

## Multiprocessing library

Let's take a look of the basic example of using multiprocessing in python.

```python
import multiprocessing
def cpu_bound_processing(data):
    pass

process = multiprocessing.Process(target=cpu_bound_processing, args=(x, y, z))
process.start()
process.join() # any process finishes but not been joined becomes zombie process.
```
with this example you can now utitlize more CPU cores to do CPU bound tasks.


### Shared Memory
when the input size is large, it is in-efficient to make copies of the object to be passed to each sub-process.  In this case, we should create one shared memory block of the original large object. 

Python have some capabilities build-in for this.  You can use [shared_memory] module to do that.

> As mentioned above, when doing concurrent programming it is usually best to avoid using shared state as far as possible. This is particularly true when using multiple processes.
>
> However, if you really do need to use some shared data then multiprocessing provides a couple of ways of doing so.

Below is an example program that divide the image into smaller chunks and process each chunk in separate process using shared memory, then combine the result.

```python
import multiprocessing as mp
import numpy as np
from PIL import Image

def process_chunk(shared_array, start, end):
    # Access the chunk from the shared array
    chunk = shared_array[start:end]

    # Simulate image processing: invert colors
    chunk = 255 - chunk

    # Write the processed chunk back to the shared array
    shared_array[start:end] = chunk

if __name__ == '__main__':
    # Load a large image
    img = np.array(Image.open('large_image.jpg'))

    # Create a shared memory block for the image
    shared_img = mp.Array('B', img.flatten())

    # Create multiple processes
    num_processes = 4
    chunk_size = len(img.flatten()) // num_processes
    processes = []
    for i in range(num_processes):
        start = i * chunk_size
        end = (i + 1) * chunk_size
        p = mp.Process(target=process_chunk, args=(shared_img, start, end))
        processes.append(p)
        p.start()

    # Wait for all processes to finish
    for p in processes:
        p.join()

    # Convert the shared array back to an image
    result_img = np.frombuffer(shared_img.get_obj()).reshape(img.shape)
    Image.fromarray(result_img).save('processed_image.jpg')
```


### Numpy multiprocessing

Often we uses [NumPy] arrays to represent image or video data.  This is a great candidate for multiprocessing.  Everything can be done with the build-in [shared_memory] module above, but the [SharedArray] library it is a wrapper around `shm_open`, `mmap` and friends function in C with python binding to makes this more user friendly when working with numpy. 

SharedArray has a few key functions:

- `SharedArray.create(name, shape, dtype=float)` creates a shared memory array
- `SharedArray.attach(name)` attaches a previously created shared memory array to a variable
- `SharedArray.delete(name)` deletes a shared memory array, however, existing attachments remain valid


### Example

With SharedArray. 

```python

import SharedArray

def multiprocess_load_images(num_workers = 12):
    files = os.listdir("train_images")

    # this assumes each file with 1400x1200 resolution with 3 (RBG) channels
    number_of_files = len(files)
    data = SharedArray.create('data', (number_of_files, 1400, 2100, 3))

    worker_amount = int(number_of_files/num_workers)
    residual = number_of_files - (num_works * worker_amount)

    def load_images(i, n):
        # Chucking the potential inputs. this could be optmized
        to_load = files[i: i+n]
        for j, file in enumerate(to_load):
            data[i + j] = cv2.imread("train_images/" + file)
        
    processes = []
    for worker_num in range(num_workers):
        run_worker_amount = worker_amount
        if worker_num == num_workers - 1 and residual != 0:
            run_worker_amount += residual
        process = multiprocessing.Process(target=load_images, args=(worker_amount*worker_num, run_worker_amount))
        processes.append(process)
        process.start()
    
    for process in processes:
        process.join()
    
    return data

```

An example with SharedMemory directly

```python
from multiprocessing.shared_memory import SharedMemory
from multiprocessing.managers import SharedMemoryManager
from concurrent.futures import ProcessPoolExecutor, as_completed
from multiprocessing import current_process, cpu_count, Process
import numpy as np

def work_with_shared_memory(shm_name, shape, dtype):
    print(f'With SharedMemory: {current_process()=}')
    # Locate the shared memory by its name
    shm = SharedMemory(shm_name)
    # Create the np.recarray from the buffer of the shared memory
    np_array = np.recarray(shape=shape, dtype=dtype, buf=shm.buf)
    return np.nansum(np_array.val)

# Some way to create that numpy array.
np_array = ...
shape, dtype = np_array.shape, np_array.dtype
with SharedMemoryManager() as smm:
        # Create a shared memory of size np_arry.nbytes
        shm = smm.SharedMemory(np_array.nbytes)
        # Create a np.recarray using the buffer of shm
        shm_np_array = np.recarray(shape=shape, dtype=dtype, buf=shm.buf)
        # Copy the data into the shared memory
        np.copyto(shm_np_array, np_array)
        # Spawn some processes to do some work
        with ProcessPoolExecutor(cpu_count()) as exe:
            fs = [exe.submit(work_with_shared_memory, shm.name, shape, dtype)
                  for _ in range(cpu_count())]
            for _ in as_completed(fs):
                pass
```

### Huge arrays in numpy

Python is notoriously known as a memory hogger and when you need to work with large amount of data it could result in out of memory error.  One trick we can use in this case is memory mapped file in numpy. 

```python
from concurrent.futures import ProcessPoolExecutor, as_completed
from multiprocessing import Process

import numpy as np

worker, nrows, ncols = 10, 1_000_000, 100

def split_size_iter(total_size:int , num_chunks: int) -> Iterator[Tuple[int, int]]:
    ...

def print_matrix(filename, worker_index_start, worker_index_end):
    matrix = np.memmap(filename, dtype=np.float32, mode='r+', shape=(worker, nrows, ncols))
    print matrix[worker_index_start: worker_index_end]


def main():
    matrix = np.memmap('test.dat', dtype=np.float32, mode='w+', shape=(worker, nrows, ncols))
    # some code to fill this matrix

    with ProcessPoolExecutor(cworker) as exe:
        fs = [exe.submit(print_matrix, 'test.dat', start, end) 
                for start,end in split_size_iter(worker, 4)]
        for _ in as_completed(fs):
                    pass
```


## Ray

[Ray] is a powerful open source platform that makes it easy to write distributed Python programs and seamlessly scale them from your laptop to a cluster.  Ray comes with support for the `mulitprocessing.Pool` API out of the box when importing `ray.util.multiprocessing`. 


### Example

```python
import math
import random
import time

def sample(num_samples):
    num_inside = 0
    for _ in range(num_samples):
        x, y = random.uniform(-1, 1), random.uniform(-1, 1)
        if math.hypot(x, y) <= 1:
            num_inside += 1
    return num_inside

def approximate_pi_distributed(num_samples):
    from ray.util.multiprocessing.pool import Pool # NOTE: Only the import statement is changed.
    pool = Pool()
        
    start = time.time()
    num_inside = 0
    sample_batch_size = 100000
    for result in pool.map(sample, [sample_batch_size for _ in range(num_samples//sample_batch_size)]):
        num_inside += result
        
    print("pi ~= {}".format((4*num_inside)/num_samples))
    print("Finished in: {:.2f}s".format(time.time()-start))
```

## Dask

[Dask] is a Python parallel computing library geared towards scaling analytics and scientific computing workloads.

Dask is composed of two parts: Dynamic task scheduling for optimized computation and Big Data collections such as like parallel arrays, dataframes, and lists that extend common interfaces like NumPy, Pandas, or Python iterators to larger-than-memory or distributed environments, which run on top of dynamic task schedulers.


### Example

```python
# Setup the cluster and client for Dask
from dask.distributed import Client, LocalCluster

n_workers = 3
cluster = LocalCluster(n_workers=n_workers, threads_per_worker=1, memory_limit='4GB')
client = Client(cluster)
client.wait_for_workers(n_workers)

# Load dataset
from dask_ml.datasets import make_classification as make_classification_dask

X_dask, y_dask = make_classification_dask(n_samples=10000, n_features=10, n_informative=5, n_redundant=5, random_state=1, chunks=10)

# Build a model
import dask_lightgbm.core as dlgbm

dmodel = dlgbm.LGBMClassifier(n_estimators=400)
dmodel.fit(X_dask, y_dask)

# make a single prediction
row = [[2.56999479, -0.13019997, 3.16075093, -4.35936352, -1.61271951, -1.39352057, -2.48924933, -1.93094078, 3.26130366, 2.05692145]]
yhat = dmodel.predict(da.from_array(np.array(row)))
print('Prediction: %d' % yhat[0])

y_pred = dmodel.predict(X_dask, client=client)

acc_score = (y_dask == y_pred).sum() / len(y_dask)
acc_score = acc_score.compute()
print(acc_score)
```


[shared_memory]: https://docs.python.org/3/library/multiprocessing.shared_memory.html#module-multiprocessing.shared_memory

[NumPy]: https://numpy.org/
[SharedArray]: https://pypi.org/project/SharedArray/

[Ray]: https://docs.ray.io/en/latest/index.html
[Dask]: https://docs.dask.org/
