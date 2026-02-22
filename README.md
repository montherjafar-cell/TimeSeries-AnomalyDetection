# TimeSeries-AnomalyDetection

## Summary:

The goal of this project was to detect injected anomalies in time series data representing memory usage. Data was collected through running several burner IO tests, and monitoring Proportional Set Size (PSS) among other metrics to detect spikes in memory usage.

## Data Acquistion:
To collect data for this evaluation, multiple IO burner tests were ran on a Linux shell with differing parameters, to simulate ordinary fluctuations in memory workload. To do this, parameters were varied slightly between runs:

-IO Size: Varied between 75-125 MBs
-Processes: Between 3-5
-Threads: 1-3
-Sleep: Sleep time between processes was varied between 700 and 1750 milliseconds per process 

To simulate memory usage spikes, IO tests were ran with modified parameters to create artifical anomalies in the data. Modified parameters include:
-Increased process count (45-50)
-Increased thread count (20-40)
-Lowered sleep values betwen processes (15-50 ms)

Performance was monitored and recorded through [https://github.com/HSF/prmon/tree/main](prmon) with a interval of one second per snapshot, and final performance data was collected as a time metric after each run. A total of 1,115 datapoints of ordinary runs were collected, and ~80 datapoints worth of artificial anomalies were collected as three separate modified-parameter runs. All the runs were then concatenated together, with anomalies injected between them, to form the final dataset.

## Models and anomaly detection
My first instinct was to use a one-class support vector machines (OCSVM) to fit on a clean training dataset, then test on an evaluation dataset to predict outliers. However, the model didn't work for detecting anomalies, possibly because the data points in my time series were mostly flat lines of the same value that jumped up or down to other flat lines, which didnt offer enough variety for the model to be able to correctly fit and cluster the data. My second approach was to use an isolation forest algorithm, which randomly partitions data into trees. The advantage of an isolation forest algorithm is that it doesn't necessarily have to fit or build distributions on the data. However, it was ineffective, as it fails to consider the order of data points as it isolates them, which means that it isn't able to "see" spikes in ordered time series data, and thus couldn't distinguish outliers. Moreover, It was also skewed by the lack of variance in values in the training dataset, which means that most values that were ordinary yet unseen were flagged as anomalies

**Local Outlier Factor**: after the failure of the first two models, which rely on detecting global anomalies in the data, and learning the general shape of the data, I changed my approach towards finding a contextual/local anomaly detection model, as anomalies in my data were seen as massive spikes, that deviate heavily from points around them. I realized that my model would have to capture a local distribution of the points, and then select the points that deviate largely from the ones around them. I first considered using a rolling Z-score algorithm, which calculates how many standard deviations a point is from a window of points around it. However, the distribution of the window would be prone to skewing by outliers, and might ignore outliers in the same window if one is bigger than the other, or the spike is bigger than the window. I decided finally to use the Local Outlier Factor algorithm, which compares te density of each point to the density of its k-nearest neighbors. Since this is a distance based model, rather than distribution, it is less prone to outliers than rolling Z-score, and is better at catching spikes in data values. After multiple tests and hyperparameter tuning, I found 550 to be the lowest suitable number of k-nearest neighbors that would successfully detect the anomalies. The model's drawbacks will be mentioned in the next section.

## Discussion and results
The model was able to detect all the injected anomalies in the training dataset. However, some potential shortcomings or drawbacks of the model are that it wouldnt be able to capture longer, less sharp trends in the data. Moreover, if multiple spikes occur in quick succession, or in a short amount of time, the model will likely stop detecting them, and might also begin to detect false positives. On the other hand, the advantages of using LOF are that it takes little training, and can be used as a robust "plug-and-play" model to detect large spikes in real time data that occur sporadically. Given the nature of my data and the anomalies injected, it suited the task well. However, for real performance monitoring, where conditions are less stable, it might fall short.

## AI Disclosure and Use
AI was used in this evaluation purely as a tool for information retrieval in the exploration phase and debugging. All the work done, including writing code, designing data pipelines, architectural decisions and research into different models and technique was done by me without AI influence. AI was just used to help in exploration and getting up to speed, and for debugging and fixing code. Models used include Google Gemini and ChatGPT


