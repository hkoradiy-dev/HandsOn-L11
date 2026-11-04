# Hands-on L11: Report

**Name: Hardi Koradiya**

**Student ID:801484363**

**Email:hkoradiy@charlotte.edu**

---

## Seed and commands

Seed used (your student ID):801484363

The commands you ran, in order. If you deviated from the steps in the README, say where and
why.

```bash
Started the Docker Spark cluster:

docker compose up -d

Verified the running containers:

docker compose ps
Checking the container or workers if they are live or not at http://localhost:8080


Checked Python syntax

python3 -m py_compile main.py

Installed the missing NumPy dependency in all three Spark containers:

docker exec -u 0 spark-master python3 -m pip install numpy
docker exec -u 0 spark-worker-1 python3 -m pip install numpy
docker exec -u 0 spark-worker-2 python3 -m pip install numpy

Copied the updated Python file into the Spark master:

docker cp main.py spark-master:/opt/spark/work-dir/main.py

Executed Task 1:

docker exec -it spark-master /opt/spark/bin/spark-submit \
  --master spark://spark-master:7077 \
  /opt/spark/work-dir/main.py task1 /opt/spark/work-dir/shared 801484363

Executed Task 2:

docker exec -it spark-master /opt/spark/bin/spark-submit \
  --master spark://spark-master:7077 \
  /opt/spark/work-dir/main.py task2 /opt/spark/work-dir/shared 801484363

The only difference from the original instructions was installing NumPy inside the Spark containers because the initial execution failed due to a missing dependency.

```

---

## Results

### Task 1: baseline and full pipeline

Paste the two lines printed by `evaluate()` (also in `shared-folder/output/task1/metrics`),
the five predictions shown and the "saved to" line.

```
Dataset split:

Training rows: 5,659

Testing rows: 1,487

Total rows: 7,146

Actual evaluation results:
baseline                         RMSE   331.5   R2  0.116

full                             RMSE   322.0   R2  0.164

Full PipelineModel saved to: /opt/spark/work-dir/shared/models/full

=== task1 done; results under /opt/spark/work-dir/shared/output/task1 ===

```

How many training and test rows did your seed give? The slides show 5,780 and 1,366 with
seed 42; why is yours different?

- My split contains 5,659 training rows and 1,487 test rows, whereas the lecture used 5,780 training rows and 1,366 test rows with seed 42.

The difference occurs because randomSplit() uses the specified random seed to determine which records belong to the training and testing datasets. Since I used my student ID, 801484363, instead of seed 42, I obtained a different distribution of records. The total number of records remains 7,146.

### Task 2: scoring with the saved model

Paste the stage list, the RMSE of the loaded model and the five predictions.

```
The saved PipelineModel was successfully loaded using PipelineModel.load().
Loaded PipelineModel stages:
        StringIndexerModel
        OneHotEncoderModel
        VectorAssembler
        LinearRegressionModel
        26/11/04 17:32:23 WARN SparkStringUtils: Truncated the string representation of a plan since it was toolarge. This behavior can be adjusted by setting 'spark.sql.debug.maxToStringFields'.

Task 2 RMSE: 322.0
Predictions saved to: /opt/spark/work-dir/shared/output/task2/predictions

=== task2 done; results under /opt/spark/work-dir/shared/output/task2 ===


```

---

## Questions

**1. Baseline versus full.** With seed 42 the lecture gets 221.6 / 0.152 for the baseline
and 220.6 / 0.160 for the full pipeline. What did you get, and how much did 97 more
features buy? The price goes from 10 to 10,000 dollars a night (the `summary()` in the
lecture): how does that affect an RMSE measured in dollars, and why do the numbers change
from seed to seed? Name one change to the pipeline you would try first (slides "Running
Example" with the model table, and "Tuning").

- With a seed based on my ID, i have an initial baseline to work from where i see an RMSE  of $331.5 and an R2 (R squared) score of .116.

- When using all 97 extra features from the entire data pipeline, i am able to decrease this RMSE significantly by $9.5 dollars! Additionally, the R2 score goes up by 0.048, which shows an improvement over initial model. In total, this means the entire pipeline improves upon the baseline by approx 2.87%.

- the results make sense when thinking through what these 97+ features offer. With only knowing the number of bedrooms in an Airbnb listing, we don’t get much info on what kind of listing it really is. if we know its room type, what kind of property it is (like house vs apartment), and even what neighbourhood it’s located in – all crucial pieces of information for determining how much someone would pay for a given Airbnb listing.

- This data includes prices ranging from approximately $10 to $10,000 per night, and  RMSE value of $322 indicates that predictions on average have an error scale of several hundred dollars.since this metric is computed by squaring the prediction errors, larger ones will have much more impact on the overall RMSE value than smaller ones.

- Finally, our results vary significantly from one seed to the next—the choice of training/testing splits yields dramatically different models and evaluation metrics when using different datasets. One possible improvement we could attempt  might be trying out some form of regularization within your Linear Regression implementation.

**2. What is in `shared-folder/models/full`?** List its folders (`find shared-folder/models/full
-maxdepth 2`) and say what each holds. Why do we save the PipelineModel rather than the
LinearRegressionModel alone? Which stages of the saved model are fitted models and which
are plain transformers?

The shared-folder/models/full directory contains the saved fitted PipelineModel and its individual stages.

The saved model includes:

PipelineModel metadata, which stores information about the complete pipeline.

StringIndexerModel, which stores the learned category-to-index mappings for categorical columns.

OneHotEncoderModel, which stores the encoding information for categorical features.

VectorAssembler metadata, which stores the feature assembly configuration.

LinearRegressionModel, which stores the learned regression coefficients and intercept.

The model is saved as a complete PipelineModel rather than saving only the LinearRegressionModel because prediction requires the same preprocessing steps used during training.

If only the LinearRegressionModel were saved, the categorical indexing, one-hot encoding, and feature assembly transformations would also need to be recreated separately. Saving the complete PipelineModel ensures that the same transformations are applied consistently during scoring.

Fitted models and transformers:

Pipeline stage                  Type

StringIndexerModel              Fitted model

OneHotEncoderModel              Fitted model

VectorAssembler                 Transformer

LinearRegressionModel           Fitted model

StringIndexerModel and OneHotEncoderModel contain learned information from the training dataset. VectorAssembler is a transformer that combines the selected columns into a feature vector. LinearRegressionModel contains the learned regression parameters.



**3. Only transformers at scoring time.** Task 2 ran no `fit()`. Compare the number of
predictions written with the number of test rows: if they differ, where did the rows go,
and which parameter of which stage decides it? What would you have to change in task 2 to
score a stream of listings instead of a Parquet file (slide "Scoring a Stream")?

Task 2 does not call fit(). Instead, it loads the previously saved PipelineModel and uses transform() to generate predictions on the test dataset.

The number of predictions written should be compared with the 1,487 test rows. If the counts differ, records containing invalid or previously unseen categorical values can be removed during transformation because the StringIndexer stage uses handleInvalid="skip".

This parameter determines how invalid categorical values are handled. When set to skip, rows containing invalid values are excluded instead of being assigned a separate index.

To score a stream of incoming Airbnb listings instead of reading a Parquet file, I would use Spark Structured Streaming. I would read the incoming data using spark.readStream, apply the already-loaded PipelineModel using transform(), and write the predictions using writeStream.

This allows the scoring application to process incoming listings continuously without retraining the model. The saved model can be reused to transform new incoming records.

---

## Problems and fixes

Anything that went wrong and what resolved it. Paste the actual error message. If nothing
went wrong, say so.

1. Missing NumPy dependency :-
- During the first Task 1 execution, Spark returned the following error:

ModuleNotFoundError: No module named 'numpy'

Cause:

The Docker Spark containers did not have NumPy installed, which is required by PySpark MLlib.

Fix:

Initially, I attempted to install NumPy using: 
--- docker exec -it spark-master python3 -m pip install numpy

However, the installation failed with:

ERROR: Could not install packages due to an OSError:
[Errno 13] Permission denied: '/nonexistent' 

resolved this by installing NumPy with root permissions in all three containers:

docker exec -u 0 spark-master python3 -m pip install numpy
docker exec -u 0 spark-worker-1 python3 -m pip install numpy
docker exec -u 0 spark-worker-2 python3 -m pip install numpy

After installing NumPy successfully, I reran Task 1 and Task 2. Both tasks completed successfully.