# Problems

## Frequency Counting

1. **Frequency Counting**: Given a list of integers, return a dictionary containing the number of times each integer appears.
	- Example: `nums = [1, 2, 2, 3, 1, 1, 4]`.
	- Expected Result:
		```python
		{
		    1: 3,
		    2: 2,
		    3: 1,
		    4: 1
		}
		```
	- Solution:
		```python
		nums = [1, 2, 2, 3, 1, 1, 4]
		
		def num_frequency(input):  
			frequencies = {}  
			for num in input:  
				if num in frequencies:  
				frequencies[num] += 1  
				else:  
				frequencies[num] = 1
			
			return frequencies
		```
1. **Total Transactions Per User**: You are given a list of transaction records:
	- Example:
		```python
		transactions = [
		    ("user1", 25),
		    ("user2", 10),
		    ("user1", 40),
		    ("user3", 15),
		    ("user2", 20),
		    ("user1", 10)
		]
		```
	- Each tuple contains: `(user_id, transaction_amount)`
	- Return a dictionary containing the **total transaction amount for each user**.
	- Expected Result:
		```python
		{
		    "user1": 75,
		    "user2": 30,
		    "user3": 15
		}
		```
	- Solution:
		- What information do you need to keep track of while iterating through the transactions?
			- I need to keep track of the transaction total for each user.
		- What should your dictionary represent? For example, `user_id → ???`
			- The dictionary would be a mapping of user_id to transaction_total.
		- Walk me through what happens when you encounter the first three transactions.
			- Each user is initialized in the dictionary. Their respective transaction totals are simply the first value.
		```python
		def transaction_totals(user_transactions):
			user_totals = {}
			
			# Assuming each tuple is properly formatted
			for transaction in user_totals:
				if transaction[0] in user_transactions:
					user_totals[transaction[0]] += transaction[1]
				else:
					user_totals[transaction[0]] = transaction[1]
			
			return user_totals
		```
1. **Counting Event Records**: You have a stream of event records:
	- Example:
		```python
		events = [
		    ("login", "user1"),
		    ("purchase", "user2"),
		    ("login", "user3"),
		    ("login", "user1"),
		    ("logout", "user2"),
		    ("purchase", "user1")
		]
		```
	- Each tuple is: `(event_type, user_id)`
	- Determine **how many times each event type occurred**.
	- Expected Result:
		```python
		{
		    "login": 3,
		    "purchase": 2,
		    "logout": 1
		}
		```
	- Solution:
		```python
		def event_counts(events):
			counts = {}
			for event in events:
				if event[0] in counts:
					counts[event[0]] += 1
				else:
					counts[event[0]] = 1
			
			return counts
		```
1. **Finding Duplicates**: Given a list of user IDs:
	- Example:
		```python
		user_ids = [
		    "user1",
		    "user2",
		    "user3",
		    "user1",
		    "user4",
		    "user2"
		]
		```
	- Return a list containing the IDs that appear more than once.
	- Expected Result: `["user1", "user2"]`.
	- Solution:
		```python
		def find_duplicates(user_ids):
			seen_users = set()
			duplicates = set()
			
			for user in user_ids:
				if user in seen_users:
					duplicates.add(user) # .append() is for lists, .add() is for sets
				else:
					seen_users.add(user)
			
			return duplicates
		```
2. **Repeat Shoppers**: Find **which users performed a `"purchase"` event more than once**.
	- Example:
		```python
		events = [
		    ("user1", "login"),
		    ("user2", "login"),
		    ("user1", "purchase"),
		    ("user1", "logout"),
		    ("user2", "purchase"),
		    ("user1", "purchase")
		]
		```
	- Expected Result: `["user1"]`.
	- Solution:
		```python
		def repeat_purchasers(events):
			users = set()
			repeat_users = set()
			
			for event in events:
				if event[1].lower() == "purchase":
					if event[0] in users:
						repeat_users.add(event[0])
					else:
						users.add(event[0])
			
			return repeat_users
		```
1. **Highest-Spending User**: Imagine you're processing transaction records:
	- Example:
		```python
		transactions = [
		    ("user1", 100),
		    ("user2", 50),
		    ("user3", 200),
		    ("user1", 75),
		    ("user2", 125),
		    ("user4", 300),
		    ("user3", 50)
		]
		```
	- Each tuple is: `(user_id, transaction_amount)`
	- Return the **user who has spent the most money**, along with their total spending.
	- Expected Result: `("user4", 300)`.
	- Solution:
		```python
		def highest_spender(transactions):
			user_totals = {}
			for transaction in transactions:
				if transaction[0] in user_totals:
					user_totals[transaction[0]] += transaction[1]
				else:
					user_totals[transaction[0]] = transaction[1]
			
			current_max = 0
			current_user = None
			for user, total in user_totals.items():
				if total > current_max:
					current_user = user
					current_max = total
			
			return (current_user, current_max)
		```
1. **Find First Unique User**: Return the **first user that appears exactly once**.
	- Example:
		```python
		user_ids = [
		    "user1",
		    "user2",
		    "user3",
		    "user2",
		    "user1",
		    "user4"
		]
		```
	- Expected Result: `"user3"`.
	- Solution:
		```python
		def first_unique_user(user_ids):
			user_frequencies = {}
			for user_id in user_ids:
				if user_id in user_frequencies:
					user_frequencies[user_id] += 1
				else:
					user_frequencies[user_id] = 1
			
			for user_id in user_ids:
				if user_frequencies[user_id] == 1:
					return user_id
		```
1. **Duplicate Transactions**: You're processing transaction records:
	- Example:
		```python
		transactions = [
		    ("txn1", "user1", 100),
		    ("txn2", "user2", 50),
		    ("txn3", "user1", 75),
		    ("txn1", "user1", 100),
		    ("txn4", "user3", 200),
		    ("txn2", "user2", 50)
		]
		```
	- Each record contains: `(transaction_id, user_id, amount)`. A transaction is considered a **duplicate** if its `transaction_id` has already appeared earlier in the list.
	- Return a list of the **duplicate transaction IDs**, without duplicates in the result.
	- Expected Result: `["txn1", "txn2"]`.
	- Solution:
		```python
		def detect_dupicates(transactions):
			seen_transactions = set()
			reported_duplicates = set()
			result = list()
			
			for transaction in transactions:
				transaction_id = transaction[0] # Duplicates are defined by the transaction ID
				
				if transaction_id not in seen_transactions:
					seen_transactions.add(transaction_id)
				else:
					if transaction_id not in reported_duplicates:
						reported_duplicates.add(transaction_id)
						result.append(transaction_id)
			
			return result
		```
1. **Log Analysis**: You're given application log records:
	- Example:
		```python
		logs = [
		    ("2026-08-15 10:00", "api1", 200),
		    ("2026-08-15 10:01", "api2", 500),
		    ("2026-08-15 10:02", "api1", 500),
		    ("2026-08-15 10:03", "api3", 200),
		    ("2026-08-15 10:04", "api2", 500),
		    ("2026-08-15 10:05", "api1", 200),
		]
		```
	- Each record contains: `(timestamp, endpoint, status_code)`.
	- Return the **endpoint with the highest number of 500 responses**, along with its count.
	- Expected Result: `("api2", 2)`.
	- Solution:
		```python
		def most_500_errors(logs):
			api_500_errors = dict()
			result_api = None
			max_error_count = 0
			
			for log in logs:
				api = log[1]
				error_code = log[2]
				
				if error_code == 500:
					if api not in api_500_errors:
						api_500_errors[api] = 1
					else:
						api_500_errors[api] += 1
					
					if api_500_errors[api] > max_error_count:
						result_api = api
						max_error_count = api_500_errors[api]
			
			return (result_api, max_error_count)
		```

## Two Sum

1. Given a list of integers and a target value, return the **indices of two numbers whose sum equals the target**.
	- Example:
		```python
		nums = [2, 7, 11, 15]
		target = 9
		```
	- Expected Result: `[0, 1]`.
	- Solution:
		```python
		def two_sum(nums, target):
			positions = {}
			
			for i in range(len(nums)):
				value = nums[i]
				needed = target - value
				
				if needed in positions:
					return [positions[needed], i]
				else:
					positions[value] = i
		```

## Sliding Window Pattern

1. **Longest Sequence**: Given a list of user IDs, find the length of the **longest consecutive sequence of the same user**.
	- Example:
		```python
		user_ids = [
		    "user1",
		    "user1",
		    "user1",
		    "user2",
		    "user2",
		    "user1",
		    "user3",
		    "user3",
		    "user3",
		    "user3"
		]
		```
	- The longest consecutive sequence is: `user3 → 4 occurrences`. So return: 4.
	- Expected Result: 4
	- Solution:
		```python
		def longest_streak(user_ids):
			if len(user_ids) == 0:
				return 0
			if len(user_ids) == 1:
				return 1
			
			current_user = None
			current_streak = 1
			longest_streak = 1
			
			for i in range(1, len(user_ids)):
				current_user = user_ids[i]
				if current_user == user_ids[i - 1]:
					current_streak += 1
				else:
					current_streak = 1
				
				if current_streak > longest_streak:
					longest_streak = current_streak
				
			return longest_streak
		```
		- At the beginning of each iteration, `current_streak` represents the length of the consecutive run ending at the previous element, and `longest_streak` represents the longest run seen so far.
1. **Maximum Subarray Sum**: Find the **largest possible sum of a contiguous subarray**.
	- Example: `nums = [-2, 1, -3, 4, -1, 2, 1, -5, 4]`.
	- Expected Result: `[4, -1, 2, 1]` has a sum of 6, which is the maximum for the array.
	- Solution:
		```python
		def max_subarray_sum(nums):
			if len(nums) == 0:
				return 0
			
			current_sum = nums[0]
			max_sum = nums[0]
			
			for i in range(1, len(nums)):
				if current_sum < 0:
					current_sum = nums[i]
				else:
					current_sum += nums[i]
				
				if current_sum > max_sum:
					max_sum = current_sum
			
			return max_sum
		```
1. **3-Minute User Activity**: You're processing user activity:
	- Example:
		```python
		events = [
		    ("10:00", "user1"),
		    ("10:01", "user2"),
		    ("10:02", "user1"),
		    ("10:03", "user3"),
		    ("10:04", "user2"),
		    ("10:05", "user4"),
		    ("10:06", "user1"),
		]
		```
	- Each tuple is: `(timestamp, user_id)`. Assume the events are already **sorted chronologically**.
	- Find the **maximum number of distinct users active during any 3-minute window**.
	- Expected Result: 3
	- Solution:
		```python
		from datetime import datetime, timedelta
		
		def max_distinct_users(events):
			if not events:
				return 0
			
			left = 0
			max_width = 3
			max_distinct_users = 0
			distinct_users = dict()
			
			event_times = list()
			for event in events:
				event_times.append(datetime.strptime(event[0], "%H:%M"))
			
			for right in range(len(events)):
				if events[right][1] in distinct_users:
					distinct_users[events[right][1]] += 1
				else:
					distinct_users[events[right][1]] = 1
				
				t1 = event_times[left]
				t2 = event_times[right]
				window_width = (t2.hour * 60 + t2.minute) - (t1.hour * 60 + t1.minute)
				
				while window_width > max_width:
					distinct_users[events[left][1]] -= 1
		
					if distinct_users[events[left][1]] == 0:
						del distinct_users[events[left][1]]
					
					left += 1
					t1 = event_times[left]
					window_width = (t2.hour * 60 + t2.minute) - (t1.hour * 60 + t1.minute)
				
				if len(distinct_users) > max_distinct_users:
					max_distinct_users = len(distinct_users)
			
			return max_distinct_users
		```
		- A nested `while` loop doesn't automatically mean quadratic time complexity. If the left and right pointers each move monotonically through the input, the total work can still be O(n).
1. **Removing Duplicates**: You are given a **sorted** list of integers:
	- Example: `nums = [1, 1, 2, 2, 2, 3, 4, 4, 5]`.
	- Modify the list **in place** so that each value appears only once.
	- Expected Result: After your function runs, the beginning of the list should contain: `[1, 2, 3, 4, 5]`.
	- Solution:
		```python
		def remove_duplicates(nums):
			if not nums:
				return 0
			
			left = 0
			right = 1
			
			while right < len(nums):
				if nums[left] != nums[right]:
					left += 1
					nums[left] = nums[right]
				
				right += 1
			
			return left + 1
		```
1. **Merge Intervals**: You are given a list of intervals:
	- Example:
		```python
		intervals = [
		    [1, 3],
		    [2, 6],
		    [8, 10],
		    [9, 12]
		]
		```
	- Each interval represents: `[start_time, end_time]`.
	- Merge overlapping intervals.
	- Expected Result:
		```python
		[
		    [1, 6],
		    [8, 12]
		]
		```
	- Solution:
		```python
		def merge_intervals(intervals):
			if len(intervals) == 0:
				return []
			if len(intervals) == 1:
				return intervals
			
			# Sort the intervals fist
			intervals.sort(key = lambda x: x[0])
			
			interval_min = intervals[0][0]
			interval_max = intervals[0][1]
			final_intervals = list()
			
			for i in range(1, len(intervals)):
				current_min = intervals[i][0]
				current_max = intervals[i][1]
				
				if current_min <= interval_max:
					if current_max > interval_max:
						interval_max = current_max
				else:
					final_intervals.append([interval_min, interval_max])
					interval_min = current_min
					interval_max = current_max
			
			final_intervals.append([interval_min, interval_max])
			
			return final_intervals
		```

## Data Engineering

1. **Sessionalization**: You receive user events sorted by timestamp:
	- Example:
		```python
		events = [
		    ("user1", "10:00"),
		    ("user1", "10:05"),
		    ("user1", "10:20"),
		    ("user2", "10:21"),
		    ("user2", "10:25"),
		    ("user1", "10:30"),
		]
		```
	- Each event is: `(user_id, timestamp)`. A new **session** begins whenever a user has been inactive for **more than 10 minutes**.
	- Return a dictionary mapping each user to their number of sessions.
	- Expected Result:
		```python
		{
		    "user1": 2,
		    "user2": 1
		}
		```
	- Solution:
		```python
		from datetime import datetime
		
		def sessionize(events):
			user_sessions = dict()
			
			for event in events:
				user_id = event[0]
				event_time = datetime.strptime(event[1], "%H:%M")
				
				if user_id not in user_sessions:
					user_sessions[user_id] = [1, event_time]
				else:
					last_event_time = user_sessions[user_id][1]
					event_gap_minutes = int((event_time - last_event_time).total_seconds() / 60)
					
					if event_gap_minutes > 10:
						user_sessions[user_id][0] += 1
						
					user_sessions[user_id][1] = event_time
			
			for user_id, session_info in user_sessions.items():
				user_sessions[user_id] = session_info[0]
			
			return user_sessions
		```
1. **Latest Event Per User**: You're processing events from an application:
	- Example:
		```python
		events = [
		    ("user1", "10:05", "login"),
		    ("user2", "10:10", "purchase"),
		    ("user1", "10:15", "search"),
		    ("user3", "10:20", "login"),
		    ("user2", "10:25", "logout"),
		    ("user1", "10:30", "purchase"),
		]
		```
	- Each record is: `(user_id, timestamp, event_type)`. The events are **not necessarily sorted by timestamp**.
	- Return the **latest event for each user**.
	- Expected Result:
		```python
		{
		    "user1": ("user1", "10:30", "purchase"),
		    "user2": ("user2", "10:25", "logout"),
		    "user3": ("user3", "10:20", "login")
		}
		```
	- Solution:
		```python
		from datetime import datetime
		
		def latest_events(events):
			if len(events) == 0:
				return {}
		
			user_events = dict()
			
			for event in events:
				user_id = event[0]
				event_time = datetime.strptime(event[1], "%H:%M")
				event_type = event[2]
				
				if user_id not in user_events:
					user_events[user_id] = (user_id, event_time, event_type)
				else:
					processed_event_time = user_events[user_id][1]
					
					if event_time > processed_event_time:
						user_events[user_id] = (user_id, event_time, event_type)
			
			return user_events
		```
1. **Duplicate + Keep Last**: You're receiving customer records from multiple source systems:
	- Example:
		```python
		records = [
		    ("user1", "Alice", "alice@example.com", "10:00"),
		    ("user2", "Bob", "bob@example.com", "10:05"),
		    ("user1", "Alice Smith", "alice.smith@example.com", "10:10"),
		    ("user3", "Charlie", "charlie@example.com", "10:15"),
		    ("user2", "Robert", "robert@example.com", "10:20"),
		    ("user1", "Alice Smith", "alice.smith@new.com", "10:25"),
		]
		```
	- Each record is: `(user_id, name, email, timestamp)`. The same `user_id` can appear multiple times because different systems may have updated information.
	- Return the **latest record for each user**.
	- Expected Result:
		```python
		{
		    "user1": ("user1", "Alice Smith", "alice.smith@new.com", "10:25"),
		    "user2": ("user2", "Robert", "robert@example.com", "10:20"),
		    "user3": ("user3", "Charlie", "charlie@example.com", "10:15")
		}
		```
	- Solution:
		```python
		from datetime import datetime
		
		def latest_customer_records(records):
			if not records:
				return {}
			
			user_records = dict()
			
			for record in records:
				user_id = record[0]
				user_name = record[1]
				user_email = record[2]
				event_time = datetime.strptime(record[3], "%H:%M")
				
				if user_id not in user_records:
					user_records[user_id] = (user_id, user_name, user_email, event_time)
				elif event_time > user_records[user_id][3]:
					user_records[user_id] = (user_id, user_name, user_email, event_time)
			
			return user_records
		```
1. **Out-of-Order Events**: You now receive **events from multiple users**, but there's an additional requirement: Some events may arrive **out of order**, and some events may have exactly the same timestamp.
	- Example:
		```python
		events = [
		    ("user1", "10:10", "search"),
		    ("user2", "10:15", "login"),
		    ("user1", "10:05", "login"),
		    ("user1", "10:10", "purchase"),
		    ("user2", "10:20", "logout"),
		]
		```
	- For each user, return their latest event. If two events for the same user have the same timestamp, **the later event in the input wins**.
	- Expected Result:
		```python
		{
		    "user1": ("user1", "10:10", "purchase"),
		    "user2": ("user2", "10:20", "logout")
		}
		```
	- Solution:
		```python
		from datetime import datetime
		
		def latest_events(events):
			if not events:
				return {}
			
			user_records = dict()
			
			for event in events:
				user_id = event[0]
				event_time = datetime.strptime(event[1], "%H:%M")
				event_type = event[2]
				
				if user_id not in user_records:
					user_records[user_id] = (user_id, event_time, event_type)
				elif event_time >= user_records[user_id][1]:
					user_records[user_id] = (user_id, event_time, event_type)
			
			return user_records
		```
1. **Transaction Anomaly Detection**: You're given a sequence of transactions:
	- Example:
		```python
		transactions = [
		    ("user1", 100),
		    ("user2", 50),
		    ("user1", 200),
		    ("user3", 500),
		    ("user2", 75),
		    ("user1", 150),
		    ("user3", 100),
		]
		```
	- Each transaction is: `(user_id, amount)`. A user is considered **high-value** if their **total transaction amount exceeds 400**.
	- Return a list of the high-value users.
	- Expected Result: `["user1", "user3"]`.
	- Solution:
		```python
		def high_value_users(transactions):
			user_totals = dict()
			high_value_users = set()
			
			for transaction in transactions:
				user_id = transaction[0]
				amount = transaction[1]
				
				if user_id not in user_totals:
					user_totals[user_id] = amount
				else:
					user_totals[user_id] += amount
				
				if user_totals[user_id] > 400 and user_id not in high_value_users:
					high_value_users.add(user_id)
			
			return high_value_users
		```
1. **Data Quality Check**: You're processing records from a data pipeline:
	- Example:
		```python
		records = [
		    ("user1", "email1@example.com"),
		    ("user2", "email2@example.com"),
		    ("user3", "email3@example.com"),
		    ("user1", "email1@example.com"),
		    ("user4", "email4@example.com"),
		    ("user2", "different@example.com"),
		]
		```
	- Each record contains: `(user_id, email)`. **Each `user_id` must map to exactly one email address**.
	- Return all users whose records contain **conflicting email addresses**.
	- Expected Result: `[user2]`.
	- Solution:
		```python
		def find_email_conflicts(records):
			if not records:
				return set()
			
			user_emails = dict()
			conflicting_records = set()
			
			for record in records:
				user_id = record[0]
				email = record[1]
				
				if user_id not in user_emails:
					user_emails[user_id] = email
				elif user_emails[user_id] != email:
					conflicting_records.add(user_id)
			
			return conflicting_records
		```
1. **Joining Two Datasets**: You have two datasets:
	- Example:
		```python
		users = [
		    ("user1", "Alice"),
		    ("user2", "Bob"),
		    ("user3", "Charlie"),
		    ("user4", "David")
		]
		
		transactions = [
		    ("user2", 100),
		    ("user1", 50),
		    ("user2", 75),
		    ("user5", 200)
		]
		```
	- You need to calculate **total spending for every user**.
	- Expected Result:
		```python
		{
		    "user1": 50,
		    "user2": 175,
		    "user3": 0,
		    "user4": 0
		    # user5 appears in transactions, but not users, so it should not appear in the result.
		}
		```
	- Solution:
		```python
		def user_transaction_totals(users, transactions):
			if not users:
				return dict()
			
			user_records = dict()
			
			for user in users:
				user_id = user[0]
				
				if user_id not in user_records:
					user_records[user_id] = 0
				
			if not transactions:
				return user_records
			
			for transaction in transactions:
				user_id = transaction[0]
				amount = transaction[1]
				
				if user_id in user_records:
					user_records[user_id] += amount
			
			return user_records
		```
1. **Customer Transaction Pipeline**: You receive two datasets:
	- Example:
		```python
		customers = [
		    ("user1", "Alice"),
		    ("user2", "Bob"),
		    ("user3", "Charlie"),
		    ("user4", "David")
		]
		
		transactions = [
		    ("txn1", "user1", 100, "completed"),
		    ("txn2", "user2", 50, "completed"),
		    ("txn3", "user1", 75, "failed"),
		    ("txn4", "user2", 100, "completed"),
		    ("txn5", "user5", 500, "completed"),
		    ("txn2", "user2", 50, "completed"),
		    ("txn6", "user3", 200, "completed"),
		    ("txn7", "user2", -25, "refunded")
		]
		```
	- Each transaction is: `(transaction_id, user_id, amount, status)`. 
	- We need to produce a report containing **every customer**, with their `user_id` and total value of **completed, non-duplicate transactions**.
	- Expected Result:
		```python
		{
		    "user1": 100,
		    "user2": 150,
		    "user3": 200,
		    "user4": 0
		}
		```
	- Solution:
		```python
		def customer_transaction_report(customers, transactions):
			if not customers:
				return dict()
			
			user_records = dict()
			processed_transactions = set()
			
			for customer in customers:
				user_records[customer[0]] = 0
			
			for transaction in transactions:
				transaction_id = transaction[0]
				user_id = transaction[1]
				amount = transaction[2]
				stauts = transaction[3]
				
				if transaction_id in processed_transactions:
					continue
				
				processed_transactions.add(transaction_id)
				
				if status != "completed":
					continue
				
				if user_id not in user_records:
					continue
				
				user_records[user_id] += amount
			
			return user_records
		```
1. **API Rate Limit Detection**: You receive API requests **sorted by timestamp**:
	- Example:
		```python
		requests = [
		    ("user1", "10:00"),
		    ("user2", "10:01"),
		    ("user1", "10:02"),
		    ("user1", "10:03"),
		    ("user2", "10:04"),
		    ("user1", "10:04"),
		    ("user1", "10:05"),
		    ("user2", "10:06"),
		]
		```
	- Each request is: `(user_id, timestamp)`. A user violates the rate limit if they make **more than 3 requests within any 5-minute window**.
	- Return the set of users who violate the limit.
	- Expected Result: `["user1"]`.
	- Solution:
		```python
		from datetime import datetime
		
		def detect_rate_limit_violations(requests):
			if not requests:
				return set()
			
			user_records = dict()
			limit_violators = set()
			left = 0
			
			for right in range(len(requests)):
				user_id = requests[right][0]
				start_time = datetime.strptime(requests[left][1], "%H:%M")
				end_time = datetime.strptime(requests[right][1], "%H:%M")
				
				if user_id not in user_records:
					user_records[user_id] = 1
				else:
					user_records[user_id] += 1
				
				while ((end_time - start_time).total_seconds / 60) > 5:
					left_user = requests[left][0]
					user_records[left_user] -= 1
					
					if user_records[left_user] == 0:
						del user_records[left_user]
					
					left += 1
					start_time = datetime.strptime(requests[left][1], "%H:%M")
				
				if user_records[user_id] > 3:
					limit_violators.add(user_id)
			
			return limit_violators
		```
1. **High-Value Users (Practice Problem)**: You have two datasets:
	- Example:
		```python
		customers = [
		    ("user1", "Alice"),
		    ("user2", "Bob"),
		    ("user3", "Charlie"),
		]
		
		events = [
		    ("user1", "purchase", 100),
		    ("user1", "login", 0),
		    ("user2", "purchase", 50),
		    ("user1", "purchase", 75),
		    ("user4", "purchase", 200),
		    ("user2", "login", 0),
		]
		```
	- Each event is: `(user_id, event_type, amount)`.
	- Return the users whose **total purchase amount is greater than $100**, along with their total.
	- Expected Result:
		```python
		{
		    "user1": 175
		}
		```
	- **Approach**:
		- A mapping of users to total purchase amount needs to be maintained. A dictionary will be used to maintain this mapping. As we iterate through the customers dataset, customers will be added to the dictionary with an initial total of $0. As we iterate through the events dataset, totals for users in the dictionary will be incremented by the amount of purchase events. When a customer's total exceeds $100, they will be added to another of users whose totals exceed $100. After iterating through the events dataset, the final dictionary of high-value users will be returned.
	- **Data Structures**:
		- Dictionary 1: Unique customers to transaction totals.
		- Dictionary 2: Unique high-value customers to transaction totals.
	- **Complexity**:
		- Time: O(a + b), where 'a' is the number of customers and 'b' is the number of transactions.
		- Space: O(a).
	- **Solution**:
		```python
		def high_value_users(customers, events):
			if not customers:
				return dict()
			
			user_records = dict()
			high_value_users = dict()
			
			for customer in customers:
				user_records[customer[0]] = 0
			
			for event in events:
				user_id = event[0]
				event_type = event[1]
				amount = event[2]
				
				if event_type != purchase:
					continue
				
				if user_id not in user_records:
					continue
				
				user_records[user_id] += amount
				current_user_total = user_records[user_id]
				
				if current_user_total > 100:
					high_value_users[user_id] = current_user_total
			
			return high_value_users
		```
		- The key idea is to invalidate events using `continue` **before** doing any work. This makes the actual event-handling logic much simpler.
		- `amount` only needs to be incremented in `user_records`, then added to `high_value_users` if the `current_user_total > 100`. You don't need to increment `amount` in `high_value_users` separately.
2. **Two-Level Aggregation (Practice Problem)**: You now receive a list of transactions:
	- Example:
		```python
		transactions = [
		    ("user1", "2026-08-15", 100),
		    ("user1", "2026-08-15", 75),
		    ("user2", "2026-08-15", 50),
		    ("user1", "2026-08-16", 200),
		    ("user2", "2026-08-16", 25),
		    ("user3", "2026-08-16", 300),
		    ("user2", "2026-08-16", 100),
		]
		```
	- For each day, which user spent the most money, and how much did they spend?
	- Expected Result:
		```python
		{
		    "2026-08-15": ("user1", 175),
		    "2026-08-16": ("user3", 300)
		}
		```
	- **Approach**:
		- As we iterate through transactions, we need to keep track of a unique user's total daily transaction amount. To achieve this, I would use a composite key of (user_id, date) and a value of transaction_total. One dictionary will store user transaction totals and another dictionary will act as the leaderboard, displaying the user with the highest daily total. The daily maximum can be determined in two passes. In the first pass, user transaction totals are calculated. In the second pass, the user with the highest total for each day is calculated.
	- **Data Structures**:
		- Dictionary 1: Daily user transaction totals.
		- Dictionary 2: Daily highest user transaction total.
	- **Complexity**:
		- Time: O(n) where n is the number of transactions.
		- Space: O(m) where m is the number of unique user-day combinations.
	- **Solution**:
		```python
		from datetime import datetime
		
		def daily_transaction_leaderboard(transactions):
			if not transactions:
				return dict()
			
			user_records = dict()
			user_leaderboard = dict()
			
			for transaction in transactions:
				user_id = transaction[0]
				date = transaction[1]
				amount = transaction[2]
				
				if (user_id, date) not in user_records:
					user_records[(user_id, date)] = amount
				else:
					user_records[(user_id, date)] += amount
				
				user_daily_total = user_records[(user_id, date)]
				
				if date not in user_leaderboard:
					user_leaderboard[date] = (user_id, user_daily_total)
				else:
					current_leader_id, current_leader_amount = user_leaderboard[date]
					
					if user_daily_total > current_leader_amount:
						user_leaderboard[date] = (user_id, user_daily_total)
			
			return user_leaderboard
		```
		- This can be done efficiently in two passes, but the solution above shows how to do it in one by decomposing the tuple value of `user_leaderboard` in order to compare the current highest total against the potential highest total. Even though `current_leader_id` is not needed, deconstructing the tuple this way is standard Python syntax. You could also use `current_leader_amount = user_leaderboard[date][1]`.
3. **Deduplication With Latest Record (Practice Problem)**: You receive records from an event stream:
	- Example:
		```python
		records = [
		    ("txn1", "user1", 100, "10:00"),
		    ("txn2", "user2", 50, "10:01"),
		    ("txn1", "user1", 125, "10:05"),
		    ("txn3", "user1", 75, "10:06"),
		    ("txn2", "user2", 60, "10:03"),
		]
		```
	- Each record is: `(transaction_id, user_id, amount, timestamp)`. Records may be **duplicates or updates of the same transaction**.
	- For each `transaction_id`, we want to keep **only the latest record**.
	- Expected Result:
		```python
		{
			"user1": 200,
			"user2": 60
		}
		```
	- **Approach**:
		- For the first pass through records, we want to keep track of a transaction's most recent version. This can be accomplished by maintaining a dictionary maps a transaction_id to its record. When a transaction_id is new, it is added to the dictionary. When a duplicate transaction arrives, its timestamp is compared against the recorded transaction and the recorded transaction is updated in the dictionary if needed. Once transactions are deduplicated, a second dictionary can map users to transaction totals by totaling all unique transactions for a user, regardless of time. This would need to be done in two passes, since records need to be properly deduplicated (keeping the latest record) before calculating totals.
	- **Data Structures**:
		- Dictionary 1: Maps the transaction ID to the **latest version** of a transaction, based on timestamp.
		- Dictionary 2: Maps the user ID to the transaction total, based on the **latest version** of transactions.
	- **Complexity**:
		- Time: O(n) where n is the number of records.
		- Space: O(m) where m is the number of unique users. 
	- **Solution**:
		```python
		from datetime import datetime
		
		def deduplicate_and_total(records):
			if not records:
				return dict()
			
			latest_records = dict()
			user_totals = dict()
			
			for txn_id, user_id, amount, timestamp in records:
				# Parse the timestamp into a datetime object
				txn_time = datetime.strptime(timestamp, "%H:%M")
				
				# Check if this transaction is the latest version of the transaction ID
				if txn_id not in latest_records:
					latest_records[txn_id] = (txn_id, user_id, amount, txn_time)
					user_totals[user_id] = user_totals.get(user_id, 0) + amount # Transaction ID is not tied to user ID
				else:
					_, existing_user_id, existing_amount, existing_time = latest_records[txn_id]
					if txn_time > existing_time: # Only update the transaction if it occurred later 
						if existing_user_id == user_id:
							user_totals[user_id] = user_totals.get(user_id, 0) - existing_amount + amount
						else:
							user_totals[existing_user_id] = user_totals.get(existing_user_id, 0) - existing_amount
							user_totals[user_id] = user_totals.get(user_id, 0) + amount
						latest_records[txn_id] = (txn_id, user_id, amount, txn_time)
			
			return user_totals
		```
		- The problem can be solved in one pass, but involves carefully updating user totals when a transaction ID is updated:
			- A transaction record is only updated when the current transaction **occurs after** the currently recorded transaction.
			- If user IDs match between the two transaction records, subtract the currently recorded transaction amount and add the current transaction amount to the current total.
			- If user IDs differ between the two transaction records, subtract the currently recorded transaction amount from the currently recorded user's total. Then add the current amount to the current user's total.
			- Finally, update the transaction in `latest records`.
1. **CDC Aggregation (Practice Problem)**: Imagine you're processing a simplified **Change Data Capture (CDC)** stream for customer accounts.
	- Each record is: `(account_id, user_id, balance, timestamp, operation)`. The `operation` is either:
		- `"upsert"` — this record represents the current state of the account
		- `"delete"` — this account has been deleted
	- Example:
		```python
		records = [
		    ("acct1", "user1", 100, "10:00", "upsert"),
		    ("acct2", "user1", 200, "10:01", "upsert"),
		    ("acct3", "user2", 500, "10:02", "upsert"),
		    ("acct1", "user1", 150, "10:05", "upsert"),
		    ("acct2", "user1", 200, "10:06", "delete"),
		    ("acct3", "user2", 600, "10:07", "upsert"),
		]
		```
	- We want to maintain the **current total account balance for each user**:
		- Records can arrive out of order.
		- A later record for an account supersedes an earlier record.
		- A delete removes the account's contribution. This means the balance in the account record should be updated to zero. The account should not actually be deleted.
		- A delete can be followed by a later upsert, which recreates the account.
		- A record with an older timestamp must be ignored.
		- If timestamps are equal, the later record in the input wins.
		- Don't sort.
		- Aim for O(n) time.
	- Expected Rsult:
		```python
		{
		    "user1": 150,
		    "user2": 600
		}
		```
	- **Approach**:
		- As we iterate through the list of records, we need to maintain a mapping of account ID to a tuple containing user ID, event time, and current balance. When an upset record is found for an account, its current balance is updated accordingly. When a delete record is found for an account, its current balance is reduced to zero. An account's record in the dictionary will only be updated if the event time for the current record is greater than or equal to the recorded event time. Once the account records have been finalized, the dictionary can be used to create another dictionary that maps a user to their total balance across all accounts.
	- **Data Structures**:
		- Dictionary 1: Mapping of account ID to account record. The account record consists of user ID, event time, and current balance.
		- Dictionary 2: Mapping of user ID to total balance across all accounts.
	- **Complexity**:
		- Time: O(n + a) where n is the number of records and a is the number of accounts.
			- Since n >= a, it can be simplified to O(n).
		- Space: O(a + u), where u is the number of users.
	- **Solution**:
		```python
		from datetime import datetime
		
		def account_cdc_aggregation(records):
		    if not records:
		        return dict()
		
		    account_records = dict()
		    user_records = dict()
		
		    for account_id, user_id, balance, event_time, event_type in records:
		        timestamp = datetime.strptime(event_time, "%H:%M")
		
		        if (
		            account_id not in account_records
		            or timestamp >= account_records[account_id][1]
		        ):
		            if event_type == "delete":
		                balance = 0
		
		            account_records[account_id] = (
		                user_id,
		                timestamp,
		                balance
		            )
		
		    for account_id, (user_id, _, balance) in account_records.items():
		        if user_id not in user_records:
		            user_records[user_id] = balance
		        else:
		            user_records[user_id] += balance
		
		    return user_records
		```
		- The first if statement in the first for loop is somewhat complicated because it needs to account for a `delete` operation being the first record for a given account. In this case, the balance should be zero.
			- If we only checked for `account_id` in the if statement, an account with an initial `delete` record may not be added to the dictionary correctly. It would be added with the record balance, not zero.

## Heaps / Top-K

1. **Top K Frequent Events**: You receive application events:
	- Example:
		```python
		events = [
		    "login",
		    "purchase",
		    "login",
		    "logout",
		    "purchase",
		    "login",
		    "search",
		    "purchase",
		    "login",
		    "search"
		]
		```
	- Return the **2 most frequently occurring event types**.
	- Expected Result: `["login", "purchase"]`.
	- Solution:
		```python
		import heapq
		
		def top_k_events(events, k):
			top_events = dict()
			heap = list()
			
			for event in events:
				if event not in top_events:
					top_events[event] = 1
				else:
					top_events[event] += 1
			
			for event, frequency in top_events.items():			
				if len(heap) < k:
					heapq.heappush(heap, (frequency, event))
				elif frequency > heap[0][0]:
					heapq.heapreplace(heap, (frequency, event))
			
			return [event for frequency, event in heap]
		```
		- **Complexity Analysis**:
			- `n` = number of events
			- `m` = number of unique event types
			- `k` = number of results requested
			- Counting Frequency: O(n)
			- Processing Heap: O(m log(k))
			- Time: O(n + m log(k))
			- Space: O(m + k)
1. **Top K Pipeline Failures**: A data pipeline produces failure events:
	- Example:
		```python
		failures = [
		    ("extract", 12),
		    ("transform", 25),
		    ("load", 8),
		    ("validate", 19),
		    ("aggregate", 30),
		]
		```
	- Each tuple contains: `(stage_name, failure_count)`.
	- Wite a function that returns the **k stages with the highest failure counts**.
	- Expected Result (k = 2):
		```python
		[
		    ("aggregate", 30),
		    ("transform", 25)
		]
		```
	- **Approach**:
		- A min heap would be useful in determining the top K failures. The heap would need to be maintained such that the head of the heap represented the bottom of the top K failures. When the heap has fewer than K elements, elements are added to the heap normally. If the heap already contains K elements and the new element is greater than the minimum, remove the minimum and insert the new element. This is preferable to sorting everything when K is small because you'd be expending a lot resources to sort a relatively large number of events compared to the number of events that need to be returned.
	- **Data Structures**:
		- Heap: Determines the top K failures.
		- Dictionary: Determines failure frequency of each stage.
	- **Complexity**:
		- Time:
			- Determining the frequencies of each stage failure takes O(m) time, where m is the number of stages.
			- Processing the heap takes O(m log(k)) time, where k is the number of requested stages.
			- The heap does **not** guarantee elements to be in sorted order. It only guarantees the smallest element will be first.
			- If the problem required the final result to be sorted, the time complexity would be O(m log(k) + k log(k)).
		- Space: The space complexity is O(k).
	- Solution:
		```python
		import heapq
		
		def top_k_failures(failures, k):
		    top_failures = list()
		
		    if not failures:
		        return top_failures
		
		    for stage, frequency in failures:
		        if len(top_failures) < k:
		            heapq.heappush(top_failures, (frequency, stage))
		        elif frequency > top_failures[0][0]:
		            heapq.heapreplace(top_failures, (frequency, stage))
		
		    return top_failures
		```
		- **Important Note**: You **cannot** use `heapq.heappush(top_failures, (stage, frequency))`. Python's `heapq` compares tuples **lexicographically**. That means it would primarily order by `stage`, not `frequency`.

## Stacks / Queues

1. **Processing Nested Data**: You're processing events from a data pipeline. Each event contains a sequence of opening and closing markers representing nested transformations:
	- Example:
		```python
		events = [
		    "extract",
		    "transform",
		    "transform_end",
		    "extract_end"
		]
		```
	- A transformation can contain another transformation, so events must close in the **reverse order** in which they were opened.
		- A nested transformation must be closed before its parent transformation can be closed.
		- For example `[()]` is valid, but `[(])` is not valid.
	- Event Types:
		```
		extract        → extract_end
		transform      → transform_end
		load           → load_end
		validate       → validate_end
		```
	- Return `True` if the events represent a valid nesting structure, and `False` otherwise.
	- Expected Result: `True`.
	- **Approach**:
		- You can use a list to maintain a record of opened operations. When a closing operation is encountered, it must match the last element in the list of opened operations. If the events match, that element is removed from the list, otherwise, you return `False`. If you can iterate through the list of events without returning `False` and the list of opened operations is empty, you would return `True`.
		- You also need a dictionary to map a closing event to its corresponding opening event.
	- **Data Structures**:
		- List: Maintain an **ordered** collection of opening events. When a closing event is encountered, it must match the last element of the list.
		- Dictionary: Maps closing events to opening events.
	- **Complexity**:
		- Time: O(n) where n is the number of events.
		- Space: O(n) in the worst case, if every event is an opening event.
	- Solution:
		```python
		def validate_pipeline(events):
		    if not events:
		        return True
		    
		    closing_to_opening = {
		        "extract_end": "extract",
		        "transform_end": "transform",
		        "load_end": "load",
		        "validate_end": "validate"
		    }
		    opening_events = set(closing_to_opening.values())
		    opened_events = list()
		
		    for event in events:
		        if event in opening_events:
		            opened_events.append(event)
		        else:
		            if not opened_events:
		                return False
		            
		            opening_event = opened_events.pop()
		
		            if opening_event != closing_to_opening[event]:
		                return False
		
		    if not opened_events:
		        return True
		    else:
		        return False
		
		```
1. **Pipeline Task Scheduling**: Suppose a data pipeline receives jobs that need to be processed in the order they arrive.
	- Each job consists of: `(job_id, processing_time)`.
	- Example:
		```python
		jobs = [
		    ("job1", 5),
		    ("job2", 3),
		    ("job3", 7),
		    ("job4", 2)
		]
		```
	- Each event consists of: `(event_type, job_id, processing_time)`.
	- Example:
		```python
		events = [
		    ("add", "job1", 5),
		    ("add", "job2", 3),
		    ("complete", "job1", 5),
		    ("add", "job3", 7),
		    ("complete", "job2", 3),
		    ("add", "job4", 2),
		]
		```
	- The pipeline has a **single worker**, so only one job can be processed at a time.
	- Some jobs may be added to the queue **while processing is occurring**. The input is a chronological sequence of events.
	- At any point:
		- `"add"` adds a job to the back of the queue.
		- `"complete"` means the currently running job has finished.
		- When the worker becomes available, it takes the job at the **front of the queue**.
		- If the queue is empty when the worker becomes available, the worker remains idle until another job arrives.
	- Write a function that returns the order in which jobs are processed.
	- Expected Result:
		```python
		[
		    "job1",
		    "job2",
		    "job3"
		]
		```
		- `job 4` is still waiting in the queue and hasn't started yet. This is because `job 1` and `job 2` were completed. Then, `job 3` was started and is still running when the queue ends.
	- **Approach**:
		- A list of queued jobs can represent the problem. A stack isn't appropriate because only one job can be processed at a time. This means the first job added to the queue most be completed before subsequent jobs can be started and completed. `append()` can be used to add jobs to the queue while `pop(0)` can be used to remove a job from the queue, once the current job is complete. When a job is completed, a `current_job` variable can be set to `None`, which can signify that a job can be removed from the queue and marked as the `current_job`.
	- **Data Structures**:
		- List 1: Represents a queue of jobs that need to be processed.
		- List 2: Represents a list of jobs that have been completed.
	- **Complexity**:
		- Time: O(n), where n is the number of events.
			- The Python `deque` library should be used to create a queue. Removing the first item from a regular list is an O(n) operation because all other elements in the list need to be shifted left.
		- Space: O(m), where m is the number of jobs.
	- Solution:
		```python
		from collections import deque
		
		def process_pipeline_events(events):
		    if not events:
		        return []
		
		    queued_jobs = deque()
		    completed_jobs = []
		    current_job = None
		
		    for event_type, job_id, _ in events:
		        if event_type == "add":
		            queued_jobs.append(job_id)
		
		            if current_job is None:
		                current_job = queued_jobs.popleft()
		
		        elif event_type == "complete" and job_id == current_job:
		            completed_jobs.append(current_job)
		            current_job = None
		
		            if queued_jobs:
		                current_job = queued_jobs.popleft()
		
		    return completed_jobs
		```

## Binary Search

1. **Find The First Failed Partition**: A data pipeline processes partitions in order. Once a partition fails, **every subsequent partition is also considered failed**.
	- You receive a sorted list of partition results where:
		- 0 = successful
		- 1 = failed
	- For example: `results = [0, 0, 0, 0, 1, 1, 1, 1]`. The first failed partition is at index 4. `results` is **guaranteed to be in sorted order**.
	- Write a function that returns the **index of the first failed partition**.
	- Expected Result: 4
	- **Approach**:
		- Use a two-pointer approach to eliminate half of the results during each iteration. The pointers are `left` and `right`. Initially, `left = 0` and `right = len(results) - 1`.
		- At each iteration `mid = (left + right) // 2`.
		- When `results[mid] == 1`, `right` is updated to `mid`, eliminating all values to the right.
		- When `results[mid] == 0`, `left` is updated to `mid + 1`, eliminating all values to the left.
		- Iteration continues while `left < right`.
		- When iteration ends, you check `results[left]` to see if it's a 1. You need to use `left` because you only iterate while `left < right`.
	- **Data Structures**:
		- No dictionary or list is required. Only a `left`, `right`, and `mid` pointer are needed.
	- **Complexity**:
		- Time: O(log(n)), where n is the number of elements in the list.
		- Space: O(1)
	- Solution:
		```python
		def first_failed_partition(results):
			if not results:
				return -1
		
			left = 0
			right = len(results) - 1
		
			while left < right:
				mid = (left + right) // 2
		
				if results[mid] == 1:
					right = mid
				else:
					left = mid + 1
		
			if results[left] == 1:
				return left
			else:
				return -1
		```
1. **Earliest Valid Partition**: A data pipeline processes partitions of increasing size. You are given the processing time for each partition size. The partition sizes are implicitly **ordered from smallest to largest**.
	- Example:
		```python
		processing_times = [2, 4, 7, 11, 16, 22, 30]
		target = 10
		```
	- Find the **smallest partition size whose processing time is at least a target threshold**.
	- Expected Result: 3. `11` is the first processing time ≥ 10.
	- **Approach**:
		- Binary search can be used because the list is sorted. If `processing_times[mid] >= threshold`, then all numbers to the right are no longer candidates. Right should be updated to mid. Mid could still be the answer, which is why it shouldn't be discarded. As the algorithm progresses, left represents the smallest processing time that could be greater than or equal to the threshold.
	- **Data Structures**:
		- No data structures are needed for the problem, just tracker variables.
	- **Complexity**:
		- Time: O(log(n)), where n is the number of partitions.
		- Space: O(1)
	- Solution:
		```python
		def first_slow_partition(processing_times, threshold):
		    if not processing_times:
		        return -1
		
		    left = 0
		    right = len(processing_times) - 1
		
		    while left < right:
		        mid = (left + right) // 2
		
		        if processing_times[mid] >= threshold:
		            right = mid
		        else:
		            left = mid + 1
		
		    if processing_times[left] >= threshold:
		        return left
		    else:
		        return -1
		```
1. **Data Quality Threshold**: You have a data pipeline that processes batches of increasing size. For each batch size, you know how much memory the batch requires. The values **are sorted** because larger batches require at least as much memory as smaller batches.
	- Example:
		```python
		memory_usage = [100, 150, 220, 310, 450, 700, 1000]
		limit = 450
		```
	- Find the **largest batch index that can safely be processed** without exceeding the memory limit.
	- Expected Result: 450
	- **Approach**:
		- Binary search can be used because the list of batches is sorted. This problem is different than the previous two problems because we're looking for a value that falls at or below a threshold. if `memory_usage[mid] <= limit`, `mid` could still be the answer because the value is allowed to be the same as the threshold. When `memory_usage[mid] <= limit`, you should search to the right. Otherwise, you should search to the left. `left` and `right` represent the range of indices that could still contain the largest valid batch.
	- **Data Structures**:
		- No data structures are needed for the problem, just tracker variables.
	- **Complexity**:
		- Time: O(log(n)), where n is the number of batches.
		- Space: O(1)
	- Solution:
		```python
		def largest_safe_batch(memory_usage, limit):
		    if not memory_usage:
		        return -1
		
		    left = 0
		    right = len(memory_usage) - 1
		
		    while left < right:
		        mid = (left + right + 1) // 2
		
		        if memory_usage[mid] <= limit:
		            left = mid
		        else:
		            right = mid - 1
		
		    if memory_usage[left] <= limit:
		        return left
		
		    return -1
		```
		- Key Takeaways describes why this algorithm is slightly modified.

## Trees / Graphs

1. **Deepest Pipeline Dependency**: A data pipeline is represented as a tree of dependencies. Each node represents a pipeline task.
	- Example:
		```python
		class TaskNode:
		    def __init__(self, name):
		        self.name = name
		        self.children = []
		
		"""
		             extract
		            /       \
		      validate      transform
		        /              \
		    schema            aggregate
		"""
		```
	- The tree has the following structure:
		```python
		root = TaskNode("extract")
		
		validate = TaskNode("validate")
		transform = TaskNode("transform")
		schema = TaskNode("schema")
		aggregate = TaskNode("aggregate")
		
		root.children = [validate, transform]
		validate.children = [schema]
		transform.children = [aggregate]
		```
	- The **depth** of a node is the number of edges from the root. In the above example:
		```
		extract     → depth 0
		validate    → depth 1
		transform   → depth 1
		schema      → depth 2
		aggregate   → depth 2
		```
	- Write a function that returns the name of a task at the greatest depth.
	- Expected Result: Either `"schema"` or `"aggregate"` is acceptable because both are at depth 2.
	- Additional Requirements:
		- The tree can have an arbitrary number of children per node.
		- The tree may contain only the root.
		- If `root` is `None`, return `None`.
		- You should target **O(n)** time, where `n` is the number of nodes.
	- **Approach**:
		- Counterintuitively, BFS would be the best approach to traversing the tree. This is because BFS traverses the tree **level-by-level**, meaning the last level encountered is **necessarily the deepest level**.
		- A queue can be built with tuples containing `(node, depth)`. Then we can keep track of the deepest node we've encountered.
	- **Data Structures**:
		- A queue will be used to track a node and its associated depth in the tree.
	- **Complexity**:
		- Time: O(n), where n is the number of pipeline tasks.
		- Space: O(n) in the worst-case scenario.
	- Solution:
		```python
		from collections import deque
		
		class TaskNode:
		    def __init__(self, name):
		        self.name = name
		        self.children = []
		
		def deepest_task(root: TaskNode):
		    if not root:
		        return None
		    
		    nodes = deque()
		    nodes.append((root, 0))
		    deepest_node = root.name
		    deepest_level = 0
		
		    while nodes:
		        task, level = nodes.popleft()
		
		        if level > deepest_level:
		            deepest_level = level
		            deepest_node = task.name
		
		        for child in task.children:
		            nodes.append((child, level + 1))
		
		    return deepest_node
		```
1. **Find a Pipeline Task**: You are given the same `TaskNode` structure as the previous problem and the root of a pipeline dependency tree and a task name.
	- Example:
		```
		             extract
		            /       \
		      validate      transform
		        /              \
		    schema            aggregate
		```
	- Write a function that returns `True` if a task with the given name exists and `False` otherwise.
	- Expected Result: `True` for `"transform"`.
	- **Approach**:
		- DFS is a reasonable choice for this problem because we're not concerned with the level or depth of the pipeline dependency. We're just concerned with its existence within the tree. You can implement DFS without recursion by building a stack. When you find the target, you return True. If you never find the target, you return False.
	- **Data Structures**:
		- A stack will be used to traverse the tree using the DFS strategy.
	- **Complexity**:
		- Time: O(n), where n is the number of pipeline dependencies. This is the worst-case scenario, where every dependency is checked.
		- Space: O(n). This is the worst-case scenario, where every dependency is checked.
	- Solution:
		```python
		class TaskNode:
		    def __init__(self, name):
		        self.name = name
		        self.children = []
		
		def find_task(root: TaskNode, target):
		    if not root:
		        return False
		    
		    nodes = list()
		    nodes.append(root)
		
		    while nodes:
		        task = nodes.pop()
		
		        if task.name == target:
		            return True
		
		        for child in task.children:
		            nodes.append(child)
		
		    return False
		```
		- You don't build up the whole stack first, then traverse through it. Instead, you `pop()` a node, then add its children.
		- Starting with the `root` node, the stack momentarily becomes empty. If `root` has children, the stack is filled with those children.
		- For each child, the same procedure is followed. This ensures you're depth increases with each iteration.
		- Children will naturally stopped being added when you reach leaf nodes, so the stack won't have the chance to grow infinitely.
1. **Pipeline Dependency Depth**: A pipeline task can have multiple dependencies.
	- Example:
		```
		                 ingest
		                /      \
		          validate      clean
		           /    \         \
		       schema  quality    normalize
		```
	- Determine the **maximum dependency depth** of the pipeline where the root has a depth of 0.
	- Expected Result: 2
	- Requirements:
		- Return `-1` if `root` is `None`.
		- A single-node tree has depth `0`.
		- Each child is one level deeper than its parent.
		- The tree can have an arbitrary number of children.
	- **Approach**:
		- I would choose BFS because you'd naturally arrive at the deepest level of the tree towards the end of the iteration. The state that needs to be maintained during the iteration is the deepest level. When using DFS, you could keep track of the depth by adding tasks to the stack as tuple. One element of the tuple could be the task itself, while the other is the level, starting with 0 for the root task. When you add children, the depth for the tasks being added would be 'current_level + 1'. The same approach could be used when adding tasks to the queue when using BFS.
	- **Data Structures**:
		-   A queue will be used to track a node and its associated depth in the tree.
	- **Complexity**:
		- Time: O(n), where n is the number of pipeline dependencies. This is the worst-case scenario, where every dependency is checked.
		- Space: O(n). This is the worst-case scenario, where the queue contains many tasks simultaneously.
	- Solution:
		```python
		from collections import deque
		
		def max_pipeline_depth(root: TaskNode):
		    if not root:
		        return -1
		    
		    nodes = deque()
		    nodes.append((root, 0))
		    max_depth = 0
		
		    while nodes:
		        task, level = nodes.popleft()
		
		        if level > max_depth:
		            max_depth = level
		
		        for child in task.children:
		            nodes.append((child, level + 1))
		
		    return max_depth
		```
1. **Pipeline Dependency Validation (Cycle Detection)**: A pipeline is supposed to have a single root task, with dependencies represented as children.
	- Example:
		```
		             extract
		            /       \
		       validate    transform
		        /              \
		    schema            aggregate
		```
	- However, the pipeline configuration may contain a **cycle** due to an erroneous dependency:
		```
		             extract
		                |
		             transform
		                |
		             aggregate
		                |
		             transform   ← cycle
		```
	- Determine whether the dependency structure contains a cycle.
	- Example Input:
		```python
		dependencies = {
		    "extract": ["validate", "transform"],
		    "validate": ["schema"],
		    "transform": ["aggregate"],
		    "schema": [],
		    "aggregate": ["transform"]
		}
		```
	- Expected Result: `True`
	- **Approach**:
		- Mimic a recursive approach, where a task and its dependencies are **recursively** added to a stack. The recursive approach is mimicked using an iterator. First, the node and its dependencies (stored in an iterator) are added to the stack.
		- Before adding to the stack, the node is checked to see if it has already been visited. If it has, the iteration of the loop is skipped to avoid processing a node more than once.
		- While there are items in the stack, nodes are added to the stack and their dependencies are inspected one-by-one. If a node has no more dependencies, it is removed from the stack and added to a list of visited nodes.
		- If a node has dependencies and a dependency is also being visited, a cycle has been detected. Otherwise, the dependency's dependencies are added to the stack.
	- **Data Structures**:
		- A stack will be used to traverse the tree using the DFS strategy.
		- Two sets will be used. One will be used to keep track of notes currently being processed. The other will be used to keep track of nodes that have been fully processed.
	- **Complexity**:
		- Time: O(V + E), where V is the number of tasks (vertices) and E is the number of dependency relationships (edges).
		- Space: O(V)
	- Solution:
		```python
		def has_cycle(dependencies: dict):
		    if not dependencies:
		        return False
		
		    visiting = set()
		    visited = set()
		    stack = list()
		
		    for start in dependencies:
		        if start in visited:
		            continue
		
		        stack.append((start, iter(dependencies.get(start, []))))
		
		        while stack:
		            node, deps = stack[-1]
		
		            if node not in visiting:
		                visiting.add(node)
		
		            try:
		                dependency = next(deps)
		            except StopIteration:
		                # Finished processing this node.
		                stack.pop()
		                visiting.remove(node)
		                visited.add(node)
		                continue
		
		            if dependency in visiting:
		                return True
		
		            if dependency in visited:
		                continue
		
		            stack.append(
		                (dependency, iter(dependencies.get(dependency, [])))
		            )
		
		    return False
		```
		- Breakdown:
			1. For each parent node (`start`) in the dictionary:
				1. If `start` is in `visited`, simply continue. **This avoids duplicate processing of fully visited nodes**.
				2. Add `start` to the stack, along with an iterator containing the node's dependencies.
				3. While the stack is not empty:
					1. Look at the last element in the stack, including the node and its dependencies (in the form of an iterator).
					2. If the node is not in `visiting`, add it to `visiting`.
					3. Try to look at the next dependency in the iterator. If there are no dependencies, remove the node from the stack, remove it from `visiting`. add it to `visited`, and continue.
					4. If there was a dependency in the iterator, return `True` if it is in `visiting`. This indicates a cycle.
					5. If the dependency is in `visited`, `continue`. No further processing needs to occur for that node.
					6. Add the dependency, along with its dependencies (in the form of an iterator) to the stack.
			- The overall point of the loop is to determine if a node that we are working on is currently in the `visiting` set.
			- The iterator acts as a stateful way to keep track of a node's processing state while progressing through the graph, without needing to use arbitrary string values like `"enter"` or `"exit"` to keep track of this. When a `StopIteration` exception is raised, the node can be removed from both the stack and `visiting`, then added to `visited`.
1. **Pipeline Dependency Ordering (Topological Sorting)**: Suppose you have a data pipeline with dependencies.
	- Example:
		```python
		dependencies = {
		    "extract": [], # 'extract' isn't waiting for anything to finish.
		    "validate": ["extract"], # 'validate' is waiting for 'extract' to finish before it starts.
		    "transform": ["validate"], # 'transform' is waiting for 'validate' to finish before it starts.
		    "load": ["transform"], # 'load' is waiting for 'transform' to finish before it starts.
		}
		```
	- A task can only run **after all of its dependencies have completed**.
	- For example, a valid execution order for the above pipeline would be: `[extract, validate, transform, load]`. Each node in the pipeline only runs once all of its dependencies have run.
	- Ordering doesn't necessarily have to be unique. Multiple orderings can occur when a nodes share dependencies.
	- If the dependencies contain a cycle, return: `[]`.
	- Write a function that returns **any valid ordering** of the pipeline tasks.
	- Expected Result: `["extract", "validate", "transform", "load"]`
	- **Approach**:
		- [[#Kahn's Algorithm]]
	- **Data Structures**:
		- Dependency Count (Dictionary): Maps tasks to the number of remaining dependencies.
		- Dependents (Dictionary): Maps a task to its dependencies.
		- Queue: Tasks currently ready to execute.
	- **Complexity**:
		- Time: O(V + E), where V is the number of tasks (vertices) and E is the number of dependency relationships (edges). Each task and dependency relationship is processed once.
		- Space: O(V + E) for the dependency counts, reverse dependency mapping, queue, and result.
	- Solution:
		```python
		def pipeline_order(dependencies: dict):
		    if not dependencies:
		        return list()
		    
		    dependency_count = dict()
		    dependents = dict()
		    queued_tasks = deque()
		    result = list()
		
		    # Initialize dependency_count and dependents.
		    # dependency_count: Maps each task to the number of dependencies.
		    # dependents: Reverse mapping of dependencies.
		    for task, dependency_list in dependencies.items():
		        dependency_count[task] = len(dependency_list)
		
		        for dependency in dependency_list:
		            if dependency in dependents:
		                dependents[dependency].append(task)
		            else:
		                dependents[dependency] = [task]
		
		    # Initialize queued tasks (those with a dependency count of 0).
		    for task, count in dependency_count.items():
		        if count == 0:
		            queued_tasks.append(task)
		
		    # Process tasks in the queue.
		    while queued_tasks:
		        task = queued_tasks.popleft()
		        result.append(task)
		        
		        for dependent_task in dependents.get(task, []):
		            dependency_count[dependent_task] -= 1
		
		            if dependency_count[dependent_task] == 0:
		                queued_tasks.append(dependent_task)
		
		    if len(dependencies) != len(result):
		        return list() # Returns an empty list if there is a cycle.
		
		    return result
		```
1. **Pipeline Continuity**: Suppose you have a pipeline represented by a mapping of dependencies:
	- Example:
		```python
		dependencies = {
		    "extract": [],
		    "clean": ["extract"],
		    "validate": ["clean"],
		    "transform": ["validate"],
		    "load": ["transform"]
		}
		```
	- Determine whether a task can be reached from another task.
	- **Approach**:
		- DFS is an appropriate choice here because you're trying to trace the lineage from one task to another. DFS traverses deeply along each lineage until the target is found or the path is exhausted.
	- **Data Structures**:
		- Stack (list): Used to inspect task lineage to determine if there is a link between `start` and `target`.
		- Visited (set): Used to identify tasks within a lineage that have already been visited (avoids infinite loops caused by cycles).
	- **Complexity**:
		- Time: O(V + E), where V is the number of tasks (vertices) and E is the number of dependency relationships (edges). Each task and dependency relationship is processed once.
		- Space: O(V + E) for the dependency counts, reverse dependency mapping, queue, and result.
	- Solution:
		```python
		def can_reach(dependents, start, target):
		    if not dependents:
		        return False
		
		    stack = [start]
		    visited = set()
		
		    while stack:
		        task = stack.pop()
		
		        if task in visited:
		            continue
		
		        visited.add(task)
		
		        if task == target:
		            return True
		
		        for dependent_task in dependents.get(task, []):
		            stack.append(dependent_task)
		
		    return False
		```

## Linked Lists

1. Suppose you're given the head of a Linked List:
	- Example: `A → B → C → D → None`.  You want to **find whether a particular value exists**.
		```python
		contains(head, "C")  # True
		contains(head, "X")  # False
		```
	- This is essentially a traversal problem.
	- **Approach**:
		- Starting at the head, check if the current node's value is equal to the target value. If it is, return `True`. Otherwise, set the current node to the current node's `next` value. 
	- **Data Structures**:
		- No data structures needed, besides the input Linked List.
	- **Complexity**:
		- **Time:** O(n) — potentially inspect every node
		- **Space:** O(1) — only one pointer/reference is maintained
	- Solution:
		```python
		class Node:
		    def __init__(self, value):
		        self.value = value
		        self.next = None
		
		def contains(head: Node, target: str):
			current = head
		
			while current:
				if current.value == target:
					return True
		
				current = current.next
		
			return False
		```
2. **Reversing a Linked List**: Suppose you have a Linked List:
	- Example: `A → B → C → D → None`
	- You want to reverse the list: `D → C → B → A → None`
	- **Approach**:
		- Initialize `previous = None` and `current = head`.
		- While `current` is not `None`, save `next`, reverse `current.next`, then move `current` forward.
	- **Data Structures**:
		- No data structures needed, besides the input list head.
	- **Complexity**:
		- Time: O(n), where n is the number of nodes in the list.
		- Space: O(1), pointers need to keep track of `previous`, `current`, and `next_node`.
	- Solution:
		```python
		class Node:
		    def __init__(self, value):
		        self.value = value
		        self.next = None
		
		def reverse_list(head: Node):
		    current = head
		    previous = None
		
		    while current:
		        next_node = current.next
		        current.next = previous
		        previous = current
		        current = next_node
		
		    return previous
		```
3. **Finding The Middle Node**: Find the middle node of a Linked List.
	- Example: `A → B → C → D → E → None`
	- Expected Result: `C`
	- For an even-length list: `A → B → C → D → None`
	- Expected Result: `C`
	- **Approach**:
		- A straightforward solution would traverse the list twice—once to count the nodes and once to find the middle.
		- There's another classic **O(n) time / O(1) space** approach using two pointers:
			- `slow` moves **one node at a time**.
			- `fast` moves **two nodes at a time**.
		- This is done because, by the time `fast` reaches the end of the list, `slow` should be at the middle.
	- **Data Structures**:
		-  No data structures needed, besides the input list head.
	- **Complexity**:
		- Time: O(n), where n is the number of nodes in the list.
		- Space: O(1), pointers need to keep track of `slow` and `fast`.
	- Solution:
		```python
		class Node:
		    def __init__(self, value):
		        self.value = value
		        self.next = None
		
		def find_middle(head: Node):
		    slow = head
		    fast = head
		
		    while fast and fast.next:
		        slow = slow.next
		        fast = fast.next.next
		
		    return slow
		```
4. **Node Removal**: Remove a node from a Linked List.
	- Example: `A → B → C → D → None`.
	- After removing `C`: `A → B → D → None`.
	- **Approach**:
		- In order to remove a node from a Linked, you need to change `current.next` to `current.next.next` for the node before the node you want to remove.
	- **Data Structures**:
		-  No data structures needed, besides the input list head.
	- **Complexity**:
		- Time: O(n), where n is the number of nodes in the list.
		- Space: O(1), pointers need to keep track of `slow` and `fast`.
	- Solution:
		```python
		def remove_node(head: Node, target: str):
		    if not head:
		        return None
		
		    if head.value == target:
		        return head.next
		
		    current = head
		
		    while current.next:
		        if current.next.value == target:
		            current.next = current.next.next
		            return head
		
		        current = current.next
		
		    return head
		```

## Dynamic Programming (DP)

1. **Pipeline Processing Cost**: Suppose you have jobs with processing times:
	- Example: `[2, 7, 3, 9, 4]`
	- Jobs need to be processed **sequentially**. You cannot process two adjacent jobs.
	- Your goal is to **maximize** the total processing time you can process.
	- For example:
		```
		Jobs:       2   7   3   9   4
		             ↓       ↓       ↓
		Choice:      ✓       ✓       ✓
		
		Total = 2 + 3 + 4 = 9
		```
		- Another possibility is `Total = 7 + 9 = 16`
	- This is a DP problem because when we're deciding whether to process job `i`, we have two choices:
		- **Take it**: Then we cannot take job `i - 1`.
		- **Skip it**: Then we can use the best result through job `i - 1`.
	- The important DP question becomes: What is the best answer for the first `i` jobs?
		- For example, `dp[0] = 2` and `dp[1] = 7`.
		- `dp[2]` is a little bit more complicated:
			```
			dp[2] = max(dp[1], dp[0] + 3)
			      = max(7, 5)
			      = 7
			```
	- This represents a fundamental DP pattern: At each position, compare the best solution **if we skip the current item** with the best solution **if we take the current item**.
	- Generally speaking:
		```
		dp[i] = max(
			dp[i - 1],          # skip current job
			dp[i - 2] + jobs[i] # take current job
		)
		```
	- Suboptimal Solution:
		```python
		def max_processing_time(jobs):
		    if not jobs:
		        return 0
		
		    if len(jobs) == 1:
		        return jobs[0]
		
		    dp = [0] * len(jobs)
		
		    dp[0] = jobs[0]
		    dp[1] = max(jobs[0], jobs[1])
		
		    for i in range(2, len(jobs)):
		        dp[i] = max(dp[i - 1], dp[i - 2] + jobs[i])
		
		    return dp[-1]
		```
		- `dp[i]` represents the maximum total processing time we can obtain from the first `i + 1` jobs, while never processing two adjacent jobs.
		- That "from the first `i + 1` jobs" part is important because `dp[i]` isn't just the value of job `i`.
		- For example:
			```python
			jobs = [2, 7, 3, 9]
			dp   = [2, 7, 7, 16]
			```
		- `dp[3] = 16` means: Looking at jobs `0` through `3` (`2, 7, 3, 9`), the best possible total is `16`. It comes from choosing `7 + 9 = 16`.
	- **Key DP Idea**: Define what `dp[i]` means → figure out how the current answer depends on previous answers.
	- For this problem:
		- `dp[i - 1]` = Best answer if we skip job `i`.
		- `dp[i - 2] + jobs[i]` = Best answer if we take job `i`.
		- Then choose the better one.
	- Optimal Solution:
		```python
		def max_processing_time(jobs):
		    if not jobs:
		        return 0
		
		    if len(jobs) == 1:
		        return jobs[0]
		
		    prev2 = jobs[0]
		    prev1 = max(jobs[0], jobs[1])
		
		    for i in range(2, len(jobs)):
		        current = max(prev1, prev2 + jobs[i])
		        prev2 = prev1
		        prev1 = current
		
		    return prev1
		```
		- Time: O(n), where n is the number of jobs.
		- Space: O(1)
		- This solution effectively maintains a **two-element sliding window over the DP results**.
		- This is a really useful DP optimization to recognize: If `dp[i]` only depends on a fixed number of previous states, you often don't need the entire DP array.
1. **Minimum Processing Cost**: You have a sequence of processing stages:
	- Example: `costs = [10, 15, 20, 5, 10]`
	- You can process either **one or two stages at a time**, and the cost of a move is the cost of the stage you **land on**. You start before stage `0` (stage `-1`) and need to reach the end.
	- What is the minimum cost required to reach the final stage?
	- Pattern Breakdown:
		- Stage 0 Cost: 10. You can only land on 10.
		- Stage 1 Cost: `min(25, 15) = 15`. You can go straight to 15, or go to 10, then 15.
		- Stage 2 Cost: `min(10, 15) + 20 = 10 + 20 = 30`. You take the minimum of the previous 2 stages and add the current stage.
		- Stage 3 Cost: `min(15, 30) + 5 = 15 + 5 = 20`. You take the minimum of the previous 2 stages and add the current stage.
		- Stage 4 Cost: `min(20, 30) + 10 = 20 + 10 = 30`. You take the minimum of the previous 2 stages and add the current stage.
		- Summary: `dp = [10, 15, 30, 20, 30]`. You take the minimum of the previous 2 stages and add the current stage.
	- In General: `dp[i] = min(dp[i - 1], dp[i - 2]) + costs[i]`.
	- Solution:
		```python
		def min_processing_cost(costs):
			if not costs:
				return 0
			
			if len(costs) == 1:
				return costs[0]
			
			prev2 = costs[0]
			prev1 = min(costs[0], costs[1])
			
			for i in range(2, len(costs)):
				current = min(prev1, prev2) + costs[i]
				prev2 = prev1
				prev1 = current
			
			return prev1
		```
1. **Maximum Job Profit**: Suppose there is a list of jobs.
	- Example: `jobs = [10, 20, 15, 30]`. Each job has a **profit**, and you cannot process two adjacent jobs.
	- Define: `dp[i]` = maximum profit obtainable from jobs `0...i`.
	- `dp = [10, 20, 25, 50]`. After initializing the first 2 elements, the decision tree looks like:
		- Keep the current maximum (`dp[i - 1`).
		- Add `dp[i - 2]` to the current profit.
		- Generally: `dp[i] = max(dp[i - 1], dp[i - 2] + jobs[i])`
	- Solution:
		```python
		def max_job_profit(jobs):
			if not jobs:
				return 0
			
			if len(jobs) == 1:
				return jobs[0]
			
			prev2 = jobs[0]
			prev1 = max(jobs[0], jobs[1])
			
			for i in range(2, len(jobs)):
				current = max(prev1, prev2 + jobs[i])
				prev2 = prev1
				prev1 = current
			
			return prev1
		```
1. **Counting Ways**: Consider a data pipeline where a job can be processed in chunks of either **1 GB or 2 GB**.
	- Example (5GB Job):
		```
		1 + 1 + 1 + 1 + 1
		1 + 1 + 1 + 2
		1 + 1 + 2 + 1
		1 + 2 + 1 + 1
		2 + 1 + 1 + 1
		2 + 2 + 1
		2 + 1 + 2
		1 + 2 + 2
		```
		- 5GB can be reached in a variety of ways, as shown above. In total, there are 8 different was to reach 5GB.
	- If `dp[i]` = number of ways to reach exactly `i` GB, then `dp[0] = 1` because there's only one way to reach 0GB (do nothing).
	- `dp = [1, 1, 2, 3, 5, 8]`.
	- Reasoning: `dp[i]` is the number of ways it took to reach `i - 2` plus the number of ways it took to reach `i - 1`, because you can go by increments of 1 or 2GB. `dp[i] = dp[i - 2] + dp[i - 1]`.
	- This "look at the possible **last decision**" technique is extremely useful for recognizing counting DP problems.
	- Solution:
		```python
		def count_ways(n):
		    if n == 0:
		        return 1
		    if n == 1:
		        return 1
		
		    prev2 = 1  # dp[0]
		    prev1 = 1  # dp[1]
		
		    for i in range(2, n + 1):
		        current = prev1 + prev2
		        prev2 = prev1
		        prev1 = current
		
		    return prev1
		```
1. **Maximum Non-Adjacent Revenue**: You have a sequence of daily revenues.
	- Example: `revenues = [5, 1, 8, 4, 10, 3]`. You want to select days to run a special promotion. However, **you cannot select two consecutive days**, because the promotion requires a recovery day afterward.
	- Return the **maximum total revenue** you can select.
	- `dp = [5, 5, 13, 13, 23, 23]`
	- Reasoning: `dp[i] = max(dp[i - 1], dp[i - 2] + revenues[i])`.
	- Solution:
		```python
		def max_revenue(revenues):
		    if not revenues:
		        return 0
		
		    if len(revenues) == 1:
		        return revenues[0]
		
		    prev2 = revenues[0]
		    prev1 = max(revenues[0], revenues[1])
		
		    for i in range(2, len(revenues)):
		        current = max(prev1, prev2 + revenues[i])
		        prev2 = prev1
		        prev1 = current
		
		    return prev1
		```
1. **Minimum Processing Cost**: Consider a data pipeline with jobs that have different processing costs.
	- Example: `costs = [4, 2, 7, 1, 3]`. You need to process **exactly `n` units of work**. At each step, you can process either **1 unit or 2 units**. But now each unit has a cost, and we want the **minimum total cost** to reach the end.
	- You can move 1 or 2 positions at a time, you need to determine the cheapest path to the final position.
	- `dp = [4, 2, 11, 3, 6]`
	- Solution:
		```python
		def min_processing_cost(costs):
		    if not costs:
		        return 0
		
		    if len(costs) == 1:
		        return costs[0]
		
		    prev2 = costs[0]
		    prev1 = min(costs[0], costs[1])
		
		    for i in range(2, len(costs)):
		        current = min(prev1, prev2) + costs[i]
		        prev2 = prev1
		        prev1 = current
		
		    return prev1
		```

# Key Takeaways

## Frequency Counting

- Use a dictionary to keep track of each **unique** entity, paired with its respective total.
	- If an entity exists in the dictionary, add the current associated value.
	- Otherwise, initialize the entity in the dictionary using the current associated value.
- Dictionaries are used in frequency counting because each key in a dictionary is unique **and dictionaries provide an efficient way to look up current counts so they can be incremented**.
- **Duplicate Counting**:
	- You don't need to use a dictionary to keep track of each frequency, then count which frequencies are greater than one.
	- Instead, you can use a set. As you iterate through the list, you maintain two sets:
		- `unique_values`: Add items that don't already exist within this set.
		- `duplicates`: Add items that exist within `unique values`.
- **Double Pass Method**:
	- In the "First Unique User" problem, we can iterate through the dictionary to find the first unique user because **dictionaries are ordered** in modern Python. A more robust approach is to use the **original user list** to iterate through the dictionary and return the first user with a frequency of one.
	- This makes the approach more understandable and version-agnostic, since dictionaries are only ordered in **modern Python versions**.
	- If you need information about the entire dataset before you can make a decision, it can be useful to make one pass to collect that information and another pass to use it.

## Two Sum

- Brute-Force Approach: Calculate every possible sum in the entire list.
	- This is inefficient because the amount of computation required grows rapidly with list size.
	- The approach also calculates redundant sums (i.e. 2 + 7 = 7 + 2).
- Efficient Approach:
	- Imagine:
		```python
		nums = [2, 7, 11, 15]
		target = 9
		```
	- We're looking for two values where: `value_1 + value_2 = 9`.
	- If we're currently looking at 2, we can ask: "What value would I need to pair with `2` to reach `9`?" That's `9 - 2 = 7`.
	- The problem now becomes: "Can I efficiently determine whether `7` has already appeared?"
	- Imagine processing the list from left to right:
		```
		index:  0  1   2   3
		value:  2  7  11  15
		```
		- At index 0: `value = 2` and `needed = 9 - 2 = 7`.
		- Have we seen 7 yet? No.
		- At index 1: `value = 7` and `needed = 9 - 7 = 2`.
		- Have we seen 2 yet? Yes. At index 0.
		- Therefore: `[0, 1]`
	- A dictionary can be used to maintain the value-to-index mapping. At each iteration:
		- The value-to-index mapping would be updated. Value-to-index makes more sense than index-to-value because we want to look up a number's index using the dictionary, not the other way around. The value of each key represents **the last known location** if duplicates arise.
		- The `needed` value would be calculated, then checked against the dictionary.
		- If it's found, return the current index and the found index.
	- **Summary**: To avoid repeated scans, you keep track of what you've found, where you've found it, and what you need during each iteration.

## Sliding Window Pattern

- **Longest Sequence Problem**:
	- While a dictionary would be useful for keeping track of users and their longest streaks, it's not necessary because you can simply keep track of the current longest streak and the associated user. For each iteration, you would track:
		- Current User
			- If same as previous_user, increment streak.
		- Current Streak
			- Updated when current_user = previous_user
			- Reset when current user != previous_user
		- Longest Streak
			- Updated according to current_streak whe current_user != previous_user
	- These values would be updated as you iterated through the list, then the final value would be returned.
- **Maximum Subarray Sum Problem**:
	- Brute-Force Approach: Find every possible sum of a contiguous subarray, then pick the largest one.
	- Reasoning:
		- If the sum of the subarray you're currently carrying is **negative**, can including that negative sum ever make a future maximum-sum subarray better?
			- No. If all numbers in the array were negative, the largest possible subarray sum would simply be the smallest negative number (absolute value). If there were any positive numbers or a zero, a new subarray should be started from that position.
		- The key strategy is **starting a new subarray when the sum of the current subarray is negative**.
	- Optimized Approach:
		- Track `current_sum` and `maximum_sum` as you iterate through the array.
		- `current_sum` is the best sum of a subarray **ending at the current position**.
		- `maximum_sum` is the best sum we've seen **anywhere in the array**.
		- At each number, choose between **starting fresh with this number** or **extending the previous subarray with this number**. If `current_sum` is negative, restart; otherwise extend.
		```
		current_sum = first number
		max_sum = first number
		
		for each subsequent number:
		    if current_sum < 0:
		        start a new subarray at this number
		    else:
		        extend the current subarray with this number
		
		    update max_sum if current_sum is larger
		```
- **3-Minute User Activity Problem**:
	- Brute-Force Approach:
		- For every event, you could treat that event as the **start of a 3-minute window**, then scan forward until you reach events outside the window.
		- For each window you'd determine the distinct users.
		- That can require repeatedly examining the same events, giving you quadratic time complexity in the worst-case scenario.
	- Optimized Approach:
		- Because the events are already sorted by timestamp, we can maintain a window:
			```
			left -------------------- right
			 ↑                          ↑
			oldest event             newest event
			```
			- As `right` moves forward, we add the new event to the window.
			- If the new event makes the window longer than 3 minutes, we move `left` forward until the window is valid again.
			- **Don't restart the window**. Instead, think: "The window is too large → remove events from the left until it's valid again."
		- We can't use a set to keep track of unique users within the window because a user can appear more than once within a window. Instead, we need to maintain a mapping of users to number of occurrences **within the window**.
			- When an event leaves the window, decrement the associated user's count.
			- If the user's count drops to zero, remove them from the dictionary.
			- The number of keys in the dictionary determines the number of unique users.
- **Removing Duplicates Problem**:
	- A left and right pointer are used to establish a window that looks for unique numbers in a **sorted** list.
	- The key idea is **not to delete** duplicates, but to replace them with the next **unique** number in the list.
	- Left Pointer: Position where the **next unique value should be written**.
	- Right Pointer: Scans through the list looking for the next unique value.
	- Example:
		```python
		nums = [1, 1, 2, 2, 2, 3, 4, 4, 5]
		```
		- Initially, `left = 0` and `right = 1`.
		- `nums[left]` and `nums[right]` are both `1`, so we don't need another `1`.
		- Move right: `left = 0` and `right = 2`
		- `nums[left]` and `nums[right]` are different now. This means we've found a new unique value.
		- Move `left` forward and **write the new value there**: `nums[left] = nums[right]`.
		- Eventually, the **beginning** of the list only contains unique values, while the right contains duplicates.
	- This is why the sorted property is so valuable: **all duplicates are adjacent**, so we only need to compare the current value against the last unique value.

## Data Engineering

- **Sessionalization Problem**:
	- Using a dictionary is the correct approach, but the dictionary needs to associate user with their session count **and last event timestamp**:
		```python
		user → [last_event_time, session_count]
		```
	- When we encounter another event for `user1`, we calculate: `current_time - last_event_time`, then decide whether to increment the session count.
	- Since the input is **already sorted by timestamp**, it doesn't need to be sorted by `user_id`. This would destroy the **global** chronological order needed to determine if a session is still valid.
	- Events can be processed in their existing order while maintaining `last_event_time` separately for each user. That gives us a true O(n) processing pass.

## Heaps / Top-K

- **Top K Frequent Events Problem**:
	- A dictionary can tells us **how often each event occurred**, but it doesn't by itself give us the **top 2**.
	- For example:
		```python
		{
		    "login": 10,
		    "purchase": 8,
		    "search": 7,
		    "logout": 2,
		    "signup": 1
		}
		```
	- You still need a way to detect the top K frequencies.
	- Straightforward Approach:
		- Sort the dictionary by frequency after it's finalized. The leads to logarithmic time complexity, specifically O(m log(m)).
		- For small dictionaries, where the total number of unique events is not much greater than K, this works.
		- However, for large dictionaries where the total number of unique events is much greater than K, this is very inefficient. For example, if there were 1,000 unique events and you only needed the top 5, it would be very inefficient to sort all 1,000 events just to get the top 5.
	- Optimized Approach:
		- We want a data structure that lets us:
			1. Add a candidate.
			2. Quickly identify the **smallest frequency among our current top K**.
			3. Remove that smallest candidate when we have more than K.
		- This gives us:
			- Counting: O(n)
			- Heap processing: O(m log(k)), where m is the number of unique events.
			- Space: O(m + k)
		- A **min-heap** is used to track the top K frequencies because it will remove the **smallest** value in the heap when it becomes full. Think of it sort of like a cache eviction policy.
		- A min-heap gives us the smallest element in **O(1)** time at the root, while insertion/removal takes **O(log k)**.
	- Mental Model:
		- Top K Largest: Keep a **min-heap** of size K, because the smallest member of the current top K is the one most likely to be kicked out.
		- Top K Smallest: Keep a **max-heap** of size K, because the largest member of the current top K is the one most likely to be kicked out.
	- Implementation:
		- Python's `heapq` compares tuples by the first element first. Each item in the queue should therefor have the following strcucture: `(frequency, event_type)`.
		- Python compares tuples lexicographically, if two frequencies are equal, Python will compare the event strings.

## Stacks / Queues

- A Python Stack can be implemented using an ordinary list.
	- `append()` is used to add items to the stack.
	- `pop()` is used to remove items from the stack.
	- `stack[-1]` is used to inspect the element at the "top" of the stack.
- Stack / Queue Types:
	- Last In First Out (LIFO): The last item added to the stack is the first item removed.
	- First In First Out (FIFO): The first item added to the stack is the first item removed.

## Binary Search

- The Binary Search algorithm only works for **sorted lists**. The basic idea is similar to the two-pointer method used in the Sliding Window pattern.
- **Find The First Failed Partition Problem**:
	- Think of `left` and `right` as defining the range of indices that **could still be the first failed partition**.
	- Initially, `left = 0` and `right = len(results) -1`. This makes every index a candidate.
	- At each iteration: `mid = (left + right) // 2`.
	- If `results[mid] == 1`, `mid` could be the first failure, so you can't discard it. The list is sorted, so everything **to the right of `mid`** cannot be the first failure, because `mid` is already a failure. Therefore: `right = mid`.
	- If `result[mid] == 0`, `mid` cannot be the first failure, so you can discard it. The list is sorted, so everything to the left of `mid` is also potentially `0`, the first failure must be **to the right**. Therefore: `left = mid + 1`.
	- Overall Pattern:
		- `results[mid] == 1`:
			- Keep mid
			- Search left
			- `right = mid`
		- `results[mid] == 0`:
			- Discard mid
			- Search right
			- `left = mid + 1`
	- A `0` being to the left of `1` doesn't automatically qualify it as the first failure, because the first `1` could be at index 0. A cleaner way to think about the goal is: Find the **leftmost index whose value is 1**. This is the standard "find first occurrence" binary-search pattern.
	- Guiding Principle: `left` and `right` represent the range containing all remaining possible answers. The answer, if one exists, is always somewhere between `left` and `right`.
	- Complexity:
		- Time: O(log(n)), where n is the number of elements in the list.
		- Space: O(1). There's no need to keep track of anything with a list or a dictionary.
- **Data Quality Threshold Problem**:
	- Since this algorithm looks for the **rightmost** qualifying value instead of the leftmost qualifying value, using `mid = (left + right + 1) // 2` can result in an infinite loop.
	- **Edge Case Example**:
		```python
		memory_usage = [100, 150, 220, 310, 450, 700, 1000]
		limit = 450
		```
		- Eventually, you'll get `left = 3` and `right = 4`.
		- This means `mid = (left + right) // 2 = 3`.
		- Using the **upper midpoint**, `mid = (left + right + 1) // 2 = 4`. Now, `memory_usage[4] = 450`, which is valid.
- **Find first (leftmost) value satisfying condition**:
	```python
	# ...
	mid = (left + right) // 2
	# ...
	if condition(mid):
	    right = mid
	else:
	    left = mid + 1
	```
- **Find last (rightmost) value satisfying condition**:
	```python
	# ...
	mid = (left + right + 1) // 2
	# ...
	if condition(mid):
	    left = mid
	else:
	    right = mid - 1
	```

## Trees / Graphs

- A tree is essentially a hierarchy. For example:
	```
	             extract
	            /       \
	      validate      transform
	        /              \
	    schema            aggregate
	```
	- This represents a data pipeline.
	- The **edges** represent dependencies between **nodes**. Each node represents a pipeline task.
		- In this example, the "extract" node is the parent node for the tree and has "validate" and "transform" as dependencies.
		- The "validate" node is a parent node and has a "schema" dependency.
		- The "transform" node is a parent node and has a "aggregate" dependency.
- There are two basic ways to explore a tree: Depth-First Search (DFS) and Breadth-First Search (BFS).
- **Deepest Pipeline Dependency Problem**: BFS is the best approach for this problem because it asks for the **deepest** dependency. BFS traversal ensures that the last level encountered is the deepest level.

| Problem                        | Approach                            |
| ------------------------------ | ----------------------------------- |
| Find whether a path exists     | DFS/BFS                             |
| Detect a cycle                 | DFS + `visiting`/`visited`          |
| Produce valid dependency order | Kahn's algorithm / topological sort |

### Kahn's Algorithm
- **Pipeline Dependency Ordering Problem**:
	- Topological sorting can be used to detect cycles without explicitly tracking a DFS path. This is commonly called **Kahn's algorithm**.
	- Instead of storing the actual remaining dependencies, maintain an **in-degree count**: `task → number of dependencies that haven't been processed`.
	- For:
		```python
		{
		    "extract": [],
		    "validate": ["extract"],
		    "transform": ["validate"],
		    "load": ["transform"]
		}
		```
	- We'd have:
		```
		extract   → 0
		validate  → 1
		transform → 1
		load      → 1
		```
		- Any task with a count of 0 is ready to run, so it's placed in a **queue**.
	- When a task is processed, we also need the reverse mapping to see who is waiting for the task:
		```python
		{
			"extract": ["validate"],
			"validate": ["transform"],
			"transform": ["load"]
		}
		```
	- Suppose we process `extract`. `validate` depends on `extract`, so we decrement its count: `validate: 1 → 0`. The mapping allows us to decrement dependency counts for all of extract's dependencies. In this case, it's just `"validate"`.
	- Now `validate` is ready, so we add it to the queue. Then `transform` becomes ready, then `load` becomes ready.
- Basic Process:
	1. Find all tasks with 0 dependencies.
	2. Put them in a queue.
	3. Remove a task from the queue.
	4. Add it to the result.
	5. Decrement the dependency count of every task that depends on it.
	6. Any task whose count reaches 0 goes into the queue.
	7. Repeat.
- When the algorithm is finished, if `len(result) < len(dependencies)`, it means some tasks could never be processed.
- Those remaining tasks must be part of a dependency cycle (or depend on one), so we return: `[]`.

### Depth-First Search (DFS)
- DFS means "go as deep as possible before coming back."
- Starting at `extract`:
	```
				 extract  ← start
				/
		  validate
			/
		schema  ← go all the way down
	```
- Once we reach `schema`, there are no more children, so we go back up:
	```
				 extract
				/
		  validate
			\
			schema  ← finished
	```
- Then we go back to `extract` and explore the other branch:
	```
				 extract
						   \
						  transform
							   \
							 aggregate
	```
- So one possible DFS order is:
	```
	extract
	validate
	schema
	transform
	aggregate
	```
- **Key Idea**: DFS follows one branch all the way down before exploring the next branch.
- DFS is commonly implemented with either recursion, or a **stack**. For example:
	```python
	stack = [root]
	
	while stack:
	    node = stack.pop()
	    # process node
	    # add children to stack
	```
	- DFS itself doesn't inherently require recursion. **A stack is the data structure that lets you implement DFS iteratively**.
	- The stack provides the "last thing added is the next thing explored" behavior.

### Breadth-First Search (BFS)
- BFS means "explore one level at a time."
- Starting at `extract`:
	```
	Level 0:
	             extract
	```
- Then all of its children:
	```
	Level 1:
	      validate      transform
	```
- Then all of their children:
	```
	Level 2:
	    schema          aggregate
	```
- So one possible BFS order is:
	```
	extract
	validate
	transform
	schema
	aggregate
	```
- **Key Idea**: BFS completely processes one level before moving to the next level.
- BFS is commonly implemented with a **queue**: For example:
	```python
	queue = [root]
	
	while queue:
	    node = queue.pop(0)
	    # process node
	    # add children to queue
	```
	- In Python, you'd normally use `collections.deque` rather than `pop(0)` because the former has a lower time complexity.

### DFS vs. BFS
- The easiest way to remember them:
	- DFS = Depth First. Go **down** before going across.
		- Typically implemented using a **stack (LIFO)**. Explore one path deeply before moving to the next.
	- BFS = Breath First. Go **across** before going down.
		- Typically implemented using a **queue (FIFO)**. Explore everything at the current level before going deeper.
- Visualized:
	```
	             A
	           /   \
	          B     C
	         / \   / \
	        D   E F   G
	```
	- DFS: `A → B → D → E → C → F → G`
	- BFS: `A → B → C → D → E → F → G`
- Complexity:
	- Both DFS and BFS offer Time and Space complexity of O(n), where n is the number of nodes. The Space Complexity of O(n) is the worst-case scenario.

### Graph Traversal
- `visited` (set): Nodes that have been completely processed.
- `visiting` (set): Nodes currently on the DFS path.
- When entering a node: `visiting.add(node)`
- When **completely** finished with a node:
	```python
	visiting.remove(node)
	visited.add(node)
	```
- While exploring a node's dependencies:
	```
	dependency in visiting
	    → cycle!
	```
	- This distinction is important because encountering a node in `visited` **doesn't necessarily mean there's a cycle**.
- DFS is useful because cycle detection depends on identifying whether an edge points back to a node on the current traversal path.

### Iterators
- An iterator is an object that remembers where you are while going through a sequence, and gives you the next item whenever you ask for it.
- A normal for loop already uses an iterator under the hood:
	```python
	"""
	Using a for loop
	"""
	
	dependencies = ["B", "C", "D"]
	
	for dependency in dependencies:
	    print(dependency)
	
	"""
	Using an iterator
	"""
	
	iterator = iter(dependencies)
	
	while True:
	    try:
	        dependency = next(iterator)
	        print(dependency)
	    except StopIteration:
	        break
	```
- An iterator is like a queue. The `next()` method removes the first item from the collection.
- The main advantage of an iterator is that it's **stateful**. It remembers where it left off when it was last called.
- When an iterator is empty, `next()` raises `StopIteration`.
- Iterators are useful for implementing **iterative DFS** because they help mimic the recursive nature of the typical implementation. The iterator maintains its state **across iterations of the for loop** in an iterative DFS implementation.
- Think of an iterator as a bookmark in a sequence. If you're reading multiple books at the same time, an iterator is like a bookmark that allows you to put one book down, start reading another book, then come back and pick up where you left off.

### Interview Takeaway
| Problem                 | Natural approach | Why                                                                                                       |
| ----------------------- | ---------------- | --------------------------------------------------------------------------------------------------------- |
| Deepest task            | BFS              | Traverses by level                                                                                        |
| Find task               | DFS              | Search until target found                                                                                 |
| Maximum depth           | BFS or DFS       | Either can track depth                                                                                    |
| Cycle Detection (Graph) | DFS              | You need to determine the relationship between <br>parents and children, not traverse the graph by level. |
- The important lesson isn't "always use BFS for depth" or "always use DFS for search." **Both traversals can often solve the same problem.**
- You should choose based on which makes the state and termination conditions easiest to reason about.

## Linked Lists

- A Singly-Linked List looks like: `A → B → C → D → None`. Each node in the list contains the following:
	```python
	class Node:
	    def __init__(self, value):
	        self.value = value
	        self.next = None
	```
- So if we have:
	```python
	a = Node("A")
	b = Node("B")
	c = Node("C")
	
	a.next = b
	b.next = c
	```
- The Linked List will look like:
	```
	a
	↓
	A → B → C → None
	```
- The important difference from a Python list is that the nodes aren't stored next to each other in some indexed sequence. Each node simply holds a **reference to the next node**.

## Dynamic Programming (DP)

- **Key DP Idea**: Define what `dp[i]` means → figure out how the current answer depends on previous answers.
- Useful DP Optimization Principle:  If `dp[i]` only depends on a fixed number of previous states, you often don't need the entire DP array.

# Interview Preparation Topics

| Priority     | Pattern                                     |
| ------------ | ------------------------------------------- |
| 🔴 Very high | Dictionaries / Sets                         |
| 🔴 Very high | Sliding Window / Two Pointers               |
| 🔴 Very high | Aggregation / Deduplication / Event Streams |
| 🟠 High      | Sorting / Intervals                         |
| 🟠 High      | Heaps / Top-K                               |
| 🟡 Medium    | Stacks / Queues                             |
| 🟡 Medium    | Binary Search                               |
| 🟡 Medium    | Trees                                       |
| 🟡 Medium    | Graphs                                      |
| 🟢 Lower     | Linked Lists                                |
| 🟢 Lower     | Dynamic Programming                         |

# Notepad (Practice Problems)

Note taking for practice problems. Only problems that pose a significant challenge or introduce a new algorithm will be recorded above.

1. **Approach**:
	- 
2. **Data Structures**:
	- 
3. **Complexity**:
	- Time:
	- Space:
4. **Solution**:
	```python
	```