append는 리스트 자체가 요소로 들어가고 extend는 리스트를 풀어서 넣는다.

```py
list1 = []
list1.append([1, 2, 3])
list1.append([4])
list1.append([5])
print(list1) # [[1, 2, 3], [4], [5]] 

list2 = []
list2.extend([1, 2, 3])
list2.extend([4])
list2.extend([5])
print(list2) # [1, 2, 3, 4, 5]
```
