# Amazon Product Reviews Sentiment Analysis
### Sentiment Analysis of Amazon Product Review Data

In today’s world where online retail generates a lot of data about customers, products, sales and customer reviews on each product, sentiment analysis has become a key tool for making sense of that data. This  allows companies to get key insights into their products and has led to increased revenue. Google’s [1] definition of Sentiment Analysis is “the process of computationally identifying and categorizing opinions expressed in a piece of text, especially in order to determine whether the writer's attitude towards a particular topic, product, etc. is positive, negative, or neutral.” Sentiment analysis is a rapidly emerging domain in the  field  of  Natural  Language Processing for classifying and analyzing the  human’s  sentiments, emotions and opinions  about   the   products   which   are expressed  in  the  form  of  text,  star  rating,  thumbs  up and thumbs  down. Our dataset  pertains to Amazon product reviews, which has been very useful  to customers for making informed decisions about purchasing a product in addition to helping  Amazon learn their product’s positives and negatives.


## Architecture
<img src="https://user-images.githubusercontent.com/12944490/56706666-d2196600-66e3-11e9-9683-022d63600116.png" width="100%" height="600">

## Components

```js
  import { Component } from '@angular/core';
  import { MovieService } from './services/movie.service';

  @Component({
    selector: 'app-root',
    templateUrl: './app.component.html',
    styleUrls: ['./app.component.css'],
    providers: [ MovieService ]
  })
  export class AppComponent {
    title = 'app works!';
  }
```

| Components           | Code          | Details          |
|:-------------        |:-------------:|:---------------|
| The "Ext-Streamer"   | extractor |AWS Lambda function crawls (Extracting) in this S3 bucket for new files on a fixed schedule (leveraging Amazon CloudWatch Events) and copies the new files into an interim S3 bucket. This triggers another Lambda which processes the incoming file and spits out(Streaming) chunks of JSON objects containing *n* reviews at a time which are another S3 bucket which will be consumed by Spark Streaming Job for sentiment analysis.|
| The "Analyzer"       | analyzer      |We tokenized the reviews into unigrams using space as the delimiter before matching them to the sentiment dictionary RDD. For every word in the review text, we looked-up the dictionary RDD and in case of a match, stored the corresponding rating in array. The sum of values in the array was stored as sentiment value. The review was classified as positive if the sentiment value is greater than zero, negative if the sentiment value is less than zero or alternatively neutral. The sentiment dataframe was thereafter joined with original review dataframe and stored in HDFS for visualization and analysis. |
| The "Visualizer"     | visualizer      |We have created multiple Hive tables which point to HDFS location path. The daily output of data frame is stored in staging table with unique sha_key produced using “reviewID”, “productID”, and “reviewTime”. Hive scripts load the data from staging to Master table after deleting duplicates. Master_Table is defined in ORC format for efficient querying. We will be querying using Hive QL and Spark SQL interactively to know various metrics such as sentiment metrics by Product id or Department. For visualization, we connected the Master Sentiment analysis hive table data to Qlikview along with product data listing product id and name to exhibit time series charts showing variation over years, months per year. We created a list box to filter data by product id or departments or collection of product ids that the buyer is interested in. The product demographic table is joined with Master Sentiment analysis table to get product name & department. We scheduled a batch job to load the data daily and track the sentiment.|
