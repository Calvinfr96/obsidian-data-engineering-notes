---
aliases:
  - "data_modeling_practice_questions"
tags:
  - de-study
source_notes:
  - "/Users/Apple/dea-workspace/dea-general-learning/Data Engineering and Modeling/data_modeling_practice_questions.md"
imported: 2026-09-24
---

# Data Modeling Practice Questions

Review: [[02_technical_prep/data_modeling/Data Modeling Fundamentals|Data Modeling Fundamentals]] · [[02_technical_prep/data_modeling/Data Modeling Cheat Sheet|Data Modeling Cheat Sheet]]

## Contents

- [[#Forecast Sales (Walmart)]]
- [[#Fake Reviews (Yelp)]]
- [[#Retention Rate (DropBox)]]
- [[#Likelihood of Trade (Robinhood)]]
- [[#Subscription Status (GrubHub)]]
- [[#Friend Recommendation Engine (Instagram)]]
- [[#Power User (Whatsapp)]]

## Forecast Sales (Walmart)
Walmart wants to forecast sales for each department in each store using historical data. create a data model so scientists and analysts can forecast sales for the following 2 years considering the macro economic as well as social media conditions.

### Clarifying Questions
- What problems are we trying to solve with this data?
  - What kind of decisions will the data be used to make?
  - Who is going to be using the data?
- How will the accuracy of data and metrics be tracked?
- Do we have existing sales data to look at?
- Should we anticipate schema changes?
- How far back should the historical data go?
- What kind of macro-economic conditions do we want to consider specifically?
- What kind of social media conditions do we want to consider specifically?
- Do we want to forecast average sales or total sales?
  - Sales over what time period? Weekly, Monthly, or Yearly?
  - Does each department use the same methodology to calculate total/average sales?
- Do we also want to look at returns?
  - Should refund/return data be subtracted from sales data?
- How should missing data be handled?
- Is sales data generated internally or do we rely on a third party to generate this data?
- Do we want to be able to look at top-performing items within each department?
- How frequently does the data need to be updated?
- What should the final output of the data look like?

### Goals and Metrics
- Goals:
  - Drive an increase in overall sales.
  - Drive an increase in average sales.
  - Reduce rate of returns.
- Metrics:
  - Average daily sales
  - Average weekly sales
  - Average monthly sales

### Data Model
- Entities Involved:
  - Customers
  - Departments
  - Products
  - Transaction
- Dimension Tables:
  - dim_customer
    - customer_id
    - customer_name
  - dim_product
    - product_id
    - department_id
    - product_name
    - price
  - dim_department
    - department_id
    - department_name
  - dim_transaction
    - transaction_id
    - transaction_amount
    - transaction_type
- Fact Tables:
  - fact_total_sales

### Solution
```text
Table dim_store_location{
  store_id int
  store_name varchar
  url varchar
  street_address varchar
  city varchar
  state varchar
  zip_code int
  country varchar
  latitude varchar
  longitude varchar
  location varchar

  Indexes {
    (store_id) [pk]
  }
}

Table dim_store_department as ds{
  department_id int
  store_id int
  department_name varchar
  store_type  varchar
  store_size  int
  start_date timestamp

  Indexes {
    (department_id) [pk]
  }
}

Ref: ds.store_id > dim_store_location.store_id

Table dim_store_contact as sc{
  contact_id int
  store_id int
  phone_number_1 int
  phone_number_2 int
  fax_1 int
  email_1 varchar
  website varchar
  open_hours int

  Indexes {
    (contact_id) [pk]
  }
}

Ref: sc.store_id > dim_store_location.store_id

Table fact_daily_sales as f1 {
  daily_sales_id int
  store_id int
  contact_id int
  department_id int 
  store_state varchar
  store_size int
  date timestamp
  store_sales int
  total_operating_hours int
  is_holiday varchar
  Promotional_Markdown varchar

  Indexes {
    (daily_sales_id) [pk]
  }
}

Ref: f1.store_id > dim_store_location.store_id
Ref: f1.department_id > dim_store_department.department_id
Ref: f1.contact_id > dim_store_contact.contact_id
```


## Fake Reviews (Yelp)
Yelp is facing an issue with fake reviews. The goal is to find those reviewers who are fake but also the restaurants which may have hired a bunch of fake reviewers. Create a data model that would help data scientists with this issue.

### Clarifying Questions
- What is the definition of a 'fake' review?
- Are we looking for fake reviews in a particular location?
- Are there specific types of restaurants that have their reviews flagged as fake more than others?
- What data needs to be collected?
- Who will be using this data?
- Over what time period are we analyzing fake reviews?
  - On a daily, weekly, monthly, or yearly basis?

### Goals and Metrics
- Goals:
  - Identify fake reviews.
  - Identify restaurants that are paying for fake reviews.
- Metrics:
  - total review count.
  - fake review count.

### Data Model
- Entities Involved:
  - Restaurants
  - Reviewers
- Dimension Tables:
  - dim_restaurant
    - restaurant_id
    - restaurant_name
    - restaurant_address
    - restaurant_city
    - restaurant_state
    - restaurant_zipcode
    - restaurant_country
  - dim_reviewer
    - reviewer_id
    - first_name
    - last_name
    - reviewer_join_date
- Fact Tables:
  - fact_daily_reviews
    - restaurant_id
    - reviewer_id
    - reviewer_ip_address
    - review_content
    - review_date
    - score

### Solution
```text
Table dim_users
{
  user_id varchar
  user_name varchar
  yelping_since datetime
  first_name varchar  
  last_name varchar  
  gender varchar  
  email_id varchar
  phone_number int
  birth_date date
  age int
  country_code varchar
  country varchar
  state varchar
  postal_code varchar
  Indexes {
    (user_id) [pk]
  }
}

Ref: dim_users.country_code > dim_country.country_code

Table dim_country
{
  country_code varchar
  country varchar
  state varchar
  city varchar
  postal_code varchar

  Indexes {
    (country_code) [pk]
  }
}


Table dim_restaurant
{
  restaurant_id varchar
	restaurant_name varchar
	restaurant_start_date date
  restaurant_address varchar
  restaurant_county_code varchar
  restaurant_country varchar
  restaurant_state varchar
  restaurant_city varchar
  restaurant_postal_code varchar
  latitude varchar
  longitude varchar
 
  Indexes {
    (restaurant_id) [pk]
  }
}

Ref: dim_restaurant.restaurant_county_code > dim_country.country_code

Table fact_review
{
  review_id varchar
  session_id varchar
  user_id varchar
  restaurant_id varchar
  review_date date
  review_time time
  type varchar
  score int
  tags varchar
  useful int
  summary varchar
  review_text varchar

  Indexes {
    (review_id) [pk]
  }
}

Ref: fact_review.user_id > dim_users.user_id
Ref: fact_review.restaurant_id > dim_restaurant.restaurant_id

Table fact_restaurant_attributes
{
  restaurant_id varchar
  bike_parking boolean
  restaurants_accepts_bitcoin boolean
  restaurants_accepts_credit_cards boolean
  garage_parking boolean
  street_parking boolean
  dogs_allowed boolean
  restaurants_priceRange boolean
  wheel_chair_accessible boolean
  valet_parking boolean
  parking_lot boolean

  Indexes {
    (restaurant_id) [pk]
  }

}

Ref: fact_restaurant_attributes.restaurant_id > dim_restaurant.restaurant_id
```


## Retention Rate (DropBox)
Create a Data Model to track retention rate of Dropbox users & factors affecting the churning rate.

### Clarifying Questions
- What do you mean by retention rate?
  - Retention over what time period?
- What do you mean by churn?
  - Churn over what time period?
- Have there been recent declines in retention or increases in churn?
  - Are there particular factors you want to look at that have historically caused this?

### Goals and Metrics
- Goals:
  - Increase retention.
  - Reduce churn.
- Metrics:
  - Retention rate.
  - Churn rate.

### Data Model
- Entities Involved:
  - Users
  - Tiers
- Dimension Tables:
  - dim_users
    - user_id
    - first_name
    - last_name
    - current_tier_id
  - dim_tiers
    - tier_id
    - tier_name
    - price
    - storage_allowed
    - users_allowed
    - transfer_limit
- Fact Tables:
  - fact_daily_sign_ups
    - user_id
    - start_date
    - start_tier
    - date
  - fact_daily_tier_change
    - user_id
    - previous_tier_id
    - current_tier_id
    - is_upgrade
    - is_downgrade
    - date
  - fact_daily_account_closure
    - user_id
    - current_tier_id
    - end_date

### Solution
```text
Table dim_users as user{
  user_id int
  user_name varchar
  email_address varchar
  contact int
  employment_type varchar
  user_since date
  Indexes {
    (user_id) [pk]
  }
}


Table dim_plan_features as plan{
  feature_id int
  feature_name varchar
  monthly_fee int
  storage_allowed int
  number_users_allowed int
  file_recovery_days int
  transfer_limit int
  phone_support varchar
  Indexes {
    (feature_id) [pk]
  }
}


Table fact_usage as usage{
  activity_id int
  user_id int
  activity_date date
  upload_type varchar
  upload_items varchar
  upload_size int
  file_transfer_size int
  dropbox_size_remaining int
  Indexes {
    (activity_id) [pk]
  }
}


Table fact_user_upgrades as upgrade{
  upgrade_id int
  user_id int
  feature_id int
  upgrade_date date
  Indexes {
    (upgrade_id) [pk]
  }
}


Ref:usage.user_id > user.user_id
Ref:upgrade.user_id > user.user_id
Ref:upgrade.feature_id > plan.feature_id
```


#

## Likelihood of Trade (Robinhood)
Create a Data Model to help ML algorithms predict the likelihood of a user completing a trade on the Robinhood platform.

### Clarifying Questions
- What are the specific qualifications for a completed trade?
- What type of algorithm will be used?
  - Is the goal to increase engagement?

### Goals and Metrics
- Goals:
  - Increase the likelihood of users completing a trade on the platform.

- Metrics:
  - Trade completion rate.

### Data Model
- Entities Involved:
  - Users
  - Securities
- Dimension Tables:
  - dim_users
    - user_id
    - user_name
    - city
    - state
    - country
  - dim_securities
    - security_id
    - name
    - abbreviation
    - company
- Fact Tables: 
  - fact_transaction
    - user_id
    - security_id
    - security_name
    - quantity
    - date
  - fact_browsing_session
    - session_id
    - user_id
    - security_id
    - security_name
    - date

### Solution
```text
Table dim_users as user{
  user_id int
  user_name varchar
  email_address varchar
  date_of_birth date
  contact_number int
  employment varchar
  city varchar
  country varchar
  citizenship varchar
  social_security_number varchar
  investment_experience varchar
  Indexes {
    (user_id) [pk]
  }
}


Table fact_user_trades as trade{
  trade_id int
  trade_date date
  trade_timestamp timestamp
  user_id int
  stock_name varchar
  traded_units int
  traded_value int
  Indexes {
    (trade_id) [pk]
  }
}


Table fact_browsing_sessions as browse{
  browse_id int
  session_id int
  stock_name varchar
  browse_duration int
  Indexes {
    (browse_id) [pk]
  }
}


Table fact_account_updates as update{
  update_id int
  update_date date
  update_time timestamp
  stock_add_watch_list varchar
  funds_added int
  stock_order_name varchar
  stock_order_type varchar
  stock_order_value int
  Indexes {
    (update_id) [pk]
  }
}


Table fact_user_session as session{
  session_id int
  user_id int
  session_date date
  session_start_time timestamp
  session_end_time timestamp
  update_id int
  trade_id int 
  Indexes {
    (session_id) [pk]
  }
}


Ref:session.user_id > user.user_id
Ref:session.update_id > update.update_id
Ref:session.trade_id > trade.trade_id
Ref:browse.session_id > session.session_id
Ref:trade.user_id > user.user_id
```


## Subscription Status (GrubHub)
Create a Data Model to track user’s subscription status and also tracking the dollar saved per user.

### Clarifying Questions
- What do you mean by 'dollar saved per user'?

### Goals and Metrics
- Goals:
  - Increase user subscriptions.
- Metrics:
  - user_subscriptions.

### Data Model
- Entities Involved:
  - Users
  - Subscription Tiers
- Dimension Tables:
  - dim_users
    - user_id
    - user_name
    - email_address
    - city
    - state
    - country
    - tier_id
  - dim_subscription_tiers
    - tier_id
    - name
    - features
    - price
- Fact Tables:
  - fact_user_subscriptions
    - subscription_id
    - user_id
    - tier_id
    - date
  - fact_user_transactions
    - transaction_id
    - user_id
    - tier_id
    - price

### Solution
```text
Table dim_users as user{
  user_id int
  user_first_name varchar
  user_last_name varchar
  age int
  gender varchar
  contact_detail int
  city varchar
  country varchar
  user_since date
  Indexes {
    (user_id) [pk]
  }
}


Table fact_subscription_status as subs{
  sub_status_id int
  user_id int
  status_start_date date
  status_end_date date
  subscription_status varchar
  Indexes {
    (sub_status_id) [pk]
  }
}


Table fact_transactions as trans{
  transaction_id int
  transaction_date date
  transaction_time timestamp
  user_id int
  sub_status_id int
  order_actual_amount int
  order_paid_amount int
  amount_saved int
  Indexes {
    (transaction_id) [pk]
  }
}


Ref: trans.user_id > user.user_id
Ref: trans.sub_status_id > subs.sub_status_id
```


## Friend Recommendation Engine (Instagram)
Create a Data Model to allow scientists come up with a friend recommendation engine.

### Clarifying Questions
- What kind of parameters will be used to recommend friends?

### Goals and Metrics
- Goals:
  - Increase user engagement rate.

- Metrics:
  - Number of posts
  - Number of connections
  - Session time

### Data Model
- Entities Involved:
  - Users
- Dimension Tables:
  - dim_users:
    - user_id
    - username
    - first_name
    - last_name
    - age
    - email_address
    - city
    - state
    - country
- Fact Tables:
  - fact_user_session
    - session_id
    - user_id
    - session_date
    - session_start_time
    - session_end_time
  - fact_user_connection
    - connection_id
    - user_id
    - connection_user_id
    - is_active
  - fact_user_posts
    - post_id
    - user_id
    - content
  - fact_user_likes
    - user_id
    - post_id
    - comment_id
    - is_liked
  - fact_user_comments
    - comment_id
    - user_id
    - content
  
### Solution
```text
Table dim_users as user{
  user_id int
  user_first_name varchar
  user_last_name varchar
  gender varchar
  contact_detail int
  facebook_account varchar
  date_of_birth date
  Indexes {
    (user_id) [pk]
  }
}


Table fact_friends_list as friend{
  list_id int
  follow_date date
  user_id int
  friend_id int
  Indexes {
    (list_id) [pk]
  }
}


Table fact_posts as post{
  post_id int
  posting_date date
  posting_type varchar
  post_comments varchar
  post_tags varchar
  hashtags_used varchar
  Indexes {
    (post_id) [pk]
  }
}


Table fact_searches as search{
  search_id int
  search_name varchar
  profile_id_viewed int
  profile_view_duration int
  chat_id int
  Indexes {
    (search_id) [pk]
  }
}


Table fact_chats as chat{
  chat_id int
  chat_date date
  receiver_user_id int
  chat_start_time timestamp
  chat_end_time timestamp
  Indexes {
    (chat_id) [pk]
  }
}


Table fact_user_sessions as session{
  session_id int
  session_date date
  session_start_timestamp timestamp
  session_end_timestamp timestamp
  user_id int
  post_id int
  search_id int
  chat_id int
  Indexes {
    (session_id) [pk]
  }
}


Ref: session.user_id > user.user_id
Ref: friend.user_id > user.user_id
Ref: friend.friend_id > user.user_id
Ref: chat.receiver_user_id > user.user_id
Ref: session.post_id > post.post_id
Ref: session.search_id > search.search_id
Ref: session.chat_id > chat.chat_id
Ref: search.chat_id > chat.chat_id
```

## Power User (Whatsapp)
Create a Data Model that can be used to define metrics for detecting a WhatsApp Power user. What product recommendation could be created from this information?

### Clarifying Questions
- What is the definition of a Power User?

### Goals and Metrics
- Goals:
  - Increase successful product recommendations
- Metrics:
  - Recommendation count

### Data Model
- Entities Involved:
  - Users
  - Products
- Dimension Tables:
  - dim_users
    - user_id
    - facebook_account_id
    - first_name
    - last_name
    - age
    - email_address
    - city
    - state
    - country
- Fact Tables:
  - fact_user_chats
    - chat_id
    - user_id
    - member_id_list
  - fact_user_messages
    - message_id
    - chat_id
    - user_id
    - date
    - content

### Solution
```text
Table dim_users as user{
  user_id int
  user_first_name varchar
  user_last_name varchar
  contact_detail int
  recovery_email varchar
  account_type varchar
  location varchar
  language varchar
  Indexes {
    (user_id) [pk]
  }
}


Table dim_features as feature{
  feature_id int
  feature_name varchar
  feature_type varchar
  Indexes {
    (feature_id) [pk]
  }
}


Table fact_user_sessions as session{
  activity_id int
  session_id int
  user_id int
  feature_id int
  session_date date
  session_start_time timestamp
  session_end_time timestamp
  device_type varchar
  os_version varchar
  browser_type varchar
  Indexes {
    (activity_id) [pk]
  }
}


Ref: session.user_id > user.user_id
Ref: session.feature_id > feature.feature_id
```
