# Spotify Analysis – Charlie Puth’s Music Trends 🎵

## Overview  
This project analyzes Charlie Puth's music trends using Spotify API data and a combination of **AWS, Python, MySQL, and Tableau**. The data is fetched through **AWS Lambda** which interacts with the Spotify API, processed, and stored in **S3** and **RDS**. MySQL is used for cleaning, transforming, and analyzing the data stored in RDS, while Tableau creates interactive visualizations to uncover insights on track popularity, album performance, and collaboration trends.

## Technologies Used  
- **AWS Lambda (Python)**: Automates API calls to fetch and process Spotify data
- **S3 & RDS**: Stores structured data
- **MySQL**: Cleans, transforms, and analyzes data with SQL queries
- **Tableau**: Creating visualizations and dashboards for insights

## Process  
1. **Data Extraction & Storage (AWS Lambda & S3):** [View the Code - `data_ingestion.py`](./data_ingestion.py)

   - **Setup & Configuration:**
      - Attaches a Lambda Layer for `requests`.
      - Grants IAM role access to S3 (`AmazonS3FullAccess`) and logging (`AWSLambdaBasicExecutionRole`).
      - Increases timeout (max 15 mins).
      - Uses Lambda Environment Variables for client_id & client_secret.
   - **Authentication:** Retrieves an OAuth token using `client_id` and `client_secret`, encoding them in Base64, and making a POST request to Spotify’s authentication URL.
   - **Fetching Artist Information:**
      - Calls the `/search` endpoint to find the artist by name.
      - Extracts key details like artist_id, popularity, followers, and genres.
   - **Fetching Albums & Tracks:**
      - Retrieves all albums & singles via `/artists/{artist_id}/albums` end point, extracting details like album_name & release_date.
      - Iterates through paginated results to collect full data.
      - Calls `/albums/{album_id}` to get additional album details like popularity and label.
      - Extracts track information from the album response, including track_name, duration_ms, explicit, and available_market_count.
      - Calls `/tracks/{track_id}` to get additional track details like popularity and artists.
   - **Transforming & Formatting Data:**
      - Converts the extracted data into a structured format using dictionaries and lists.
      - Flattens nested data to match the CSV schema.
      - Joins multi-value fields (e.g., genres, artists) into comma-separated strings.
      - Ensures consistency in data types (e.g., converting lists into strings).
   - **Storing Data in S3**
      - Uses `csv.DictWriter` to write structured data into a CSV format.
      - Uses `boto3.client("s3").put_object()` to store the CSV file in an S3 bucket, naming it with a unique filename with a timestamp.
   - **Triggered via Input:** Accepts artist_name as an input parameter (defaults to “Charlie Puth” if not provided).

2. **Data Loading & Database Integration (AWS Lambda & RDS MySQL):** [View the Code - `data_loading.py`](./data_loading.py)
   - **Setup & Pre-requisite:**
     - Create an inital database and its table in RDS using MySQL Workbench.
     - Attaches a Lambda Layer for `requests`, `pandas`, `sqlalchemy`, and `pymysql`.
     - Grants IAM role access to S3 (`s3:GetObject`) and RDS (`rds-db:connect`).
   - **Retrieving & Reading Data from S3:** Connects to S3, reads the CSV file, and loads it into a Pandas DataFrame.
   - **Connecting to RDS MySQL:**
      - Retrieves credentials (`DB_USER`, `DB_PASSWORD`, etc.) from environment variables.
      - Establishes a connection with RDS (MySQL database) using `sqlalchemy` & `pymysql`.
   - **Loading Data into MySQL Table:** Writes DataFrame to MySQL table via `df.to_sql()`, replacing existing data if necessary.
     
3. **Database (MySQL on RDS)**
   - **Connecting MySQL Workbench to RDS:**
      - Uses RDS endpoint, username, and password to establish a direct connection.
      - Verifies access with `SHOW DATABASES;`.
   - **Cleaning & Preparing Data:**
     - Handles NULL values using `UPDATE` to fill missing data.
     - Converts data types:
       - `ALTER TABLE artist_data MODIFY COLUMN release_date DATE;`
     - Removes duplicates using DELETE with `ROW_NUMBER()`.
   - **Performing SQL Queries for Data Analysis:**
      - Identifies top tracks by popularity:
        - `SELECT track_name, popularity FROM artist_data ORDER BY popularity DESC LIMIT 10;`
      - Analyzes album releases per year:
        - `SELECT YEAR(release_date), COUNT(*) FROM artist_data GROUP BY YEAR(release_date);`
      - Filters explicit vs. non-explicit tracks, single vs. album.

5. **Visualization with Tableau:**  
   - **Setup:** Connects Tableau (paid version) to the RDS MySQL database with credentials.
   - **Building Visualizations:** 
     - Trend of Releases / Track Popularity Evolution (Line Chart)
     - Top 10 Albums by Popularity with Top Tracks (Bar Chart)
     - Top 10 Tracks by Popularity (Bar Chart)
     - Album vs. Singles Performance (Comparison Chart)
     - Track Duration vs. Popularity (Scatter Plot)
     - Collaborations & Their Impact (Pie Chart)
   
## Results  
Uncovered key trends in Charlie Puth's music releases, track popularity, and collaboration impact!
   - Charlie Puth’s music releases peaked in 2015, but hit singles like "See You Again" and "Cheating on You" continue to drive major popularity spikes despite fewer overall releases.
   - Found that hit singles drive temporary popularity spikes, while albums generally perform better overall.
   - Songs around 3 to 3.5 minutes perform best, while longer tracks see lower popularity.
   - Analyzed how collaborations influence popularity, with solo tracks often outperforming them.

**Check out the AWS Lambda functions and visualizations in this repo!**

## Links  
- [LinkedIn Post](https://www.linkedin.com/posts/hokiuchan_just-completed-my-latest-data-project-activity-7300703709288673280-vN01?utm_source=social_share_send&utm_medium=member_desktop_web&rcm=ACoAADxEAagBTXXLw3KTU9asavvuZBUpbgGoyN0)  
- [Tableau Public Dashboard](https://public.tableau.com/views/SpotifyAnalysisCharliePuthsMusicTrends1/SpotifyAnalysisCharliePuthsMusicTrends?:language=en-US&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link)  

![Spotify Analysis – Charlie Puth’s Music Trends Dashboard](./Spotify%20Analysis_Dashboard.png)

---
