
- 理解 = 、<>、 <=>、 IS NULL、 IS NOT NULL
```mysql
SELECT 10 = 10		# 1
SELECT 10 = null		# null
SELECT 10 <> null		# null
SELECT  10 <=> null 	# 0
SELECT null = null		# null
SELECT null IS NULL		# 1
SELECT 10 IS NOT NULL  # 1
SELECT 10 IS NULL		# 0
SELECT 10 != null		# null
```
- 总结对于有null参与的比较
	- <=>、 IS NULL、 IS NOT NULL ，才能返回正确的值（1/0）
	- = 、<> ，直接返回的是null