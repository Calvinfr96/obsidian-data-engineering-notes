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

## Heaps / Priority Queues

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

## Heaps / Priority Queues

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

# Notepad (Practice Problems)

Note taking for practice problems. Only problems that pose a significant challenge or introduce a new algorithm will be recorded above.

1. **Approach**:
	- For the first pass through records, we want to keep track of a transaction's most recent version. This can be accomplished by maintaining a dictionary maps a transaction_id to its record. When a transaction_id is new, it is added to the dictionary. When a duplicate transaction arrives, its timestamp is compared against the recorded transaction and the recorded transaction is updated in the dictionary if needed. Once transactions are deduplicated, a second dictionary can map users to transaction totals by totaling all unique transactions for a user, regardless of time. This would need to be done in two passes, since records need to be properly deduplicated (keeping the latest record) before calculating totals.
2. **Data Structures**:
	- 
3. **Complexity**:
	- Time: O(n) where n is the number of records.
	- Space: O(m) where m is the number of unique users. 
4. **Solution**:
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
	            user_totals[user_id] = user_totals.get(user_id, 0) + amount
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