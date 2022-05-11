---
title: Algorithm02_Priority Queue and Heap
toc: true
toc_sticky: true
toc_label: "Contents of Page"
categories:
  - Study
tags:
  - Algorithm
---

<br><br><br>

# 01. 최소 힙
## 01-1) heapq 라이브러리를 사용하지 않고 구현

```python

class PriorityQueue:
    def __init__(self) :
        self.data = []
    def push(self, value) :

        self.data.append(value)
        index = len(self.data) -1
        
        while index != 1 :
            # 마지막으로 삽입한 값이 루트노드가 되면 종료
            if self.data[index//2] > self.data[index] :
                temp = self.data[index]
                self.data[index] = self.data[index//2]
                self.data[index//2] = temp
                
                index = index//2
            else : #self.data[index//2] <= self.data[index] 
                break


    def pop(self) :
        '''
        우선순위가 가장 높은 원소를 제거합니다.
        '''
        if len(self.data) == 1:
            return
            
        # 마지막 노드를 루트 노드 자리로 이동
        self.data[1] = self.data[-1]
        self.data.pop()
        #원래있었던 루트노드 사라짐
        #[1,2,3,4]  -> [4,2,3]
        
        index = 1
        while True :
            pri = -1 #왼쪽 오른쪽 판단 변수
            # 1. 아무 자식도 없는 경우
            if len(self.data) -1 < index * 2 :
                break
            # 2. 왼쪽 자식만 있는 경우
            elif len(self.data) -1 < index *2 +1 :
                pri = index * 2
            # 3. 왼쪽, 오른쪽 모두 자식이 있는 경우
            else : 
                if self.data[index*2] <self.data[index*2 +1] :
                    pri = index *2
                else :
                    pri = index*2 +1
            
            if self.data[index] > self.data[pri] :
                temp = self.data[index]
                self.data[index] = self.data[pri]
                self.data[pri] = temp
                index = pri
            else :
                break
                

    def top(self) :
        '''
        우선순위가 가장 높은 원소를 반환합니다. 만약 우선순위 큐가 비어있다면 -1을 반환합니다.
        '''
        if len(self.data) == 1 :
            return -1
        else : 
            return self.data[1]
            
```


<br>

## 01-2) heapq 라이브러리 사용하여 구현

```python

import heapq

class PriorityQueue:
    def __init__(self) :
        self.data = []


    def push (self, value):
        heapq.heappush(self.data, value)
        
    def pop (self) :
        if len(self.data) > 0 :
            heapq.heappop(self.data)
    
    def top(self) :
        if len(self.data) == 0:
            return -1
        else :
            return self.data[0]

```

<br><br><br>


# 02. 최대 힙

```python

import heapq
class PriorityQueue:
    '''
    우선순위 큐를 힙으로 구현합니다
    '''

    def __init__(self) :
        self.data = []

    def push (self, value):
        heapq.heappush(self.data, -value)
        
    def pop (self) :
        if len(self.data) > 0 :
            heapq.heappop(self.data)
    
    def top(self) :
        if len(self.data) == 0:
            return -1
        else :
            return -self.data[0]
           
```

<br><br><br>

# 03. 절댓값 힙

```python

import heapq
class PriorityQueue:
    '''
    우선순위 큐를 힙으로 구현합니다
    '''

    def __init__(self) :
        self.data = []

    def push(self, value) :
        '''
        우선순위 큐에 value를 삽입합니다.
        '''
        heapq.heappush(self.data, (abs(value), value))

    def pop(self) :
        '''
        우선순위가 가장 높은 원소를 제거합니다.
        '''
        if len(self.data) > 0 :
            heapq.heappop(self.data)
            

    def top(self) :
        '''
        우선순위가 가장 높은 원소를 반환합니다. 만약 우선순위 큐가 비어있다면 -1을 반환합니다.
        '''
        if len(self.data) == 0 :
            return -1
        else :
            return self.data[0][1]


```

<br><br><br>

# 04. 힙정렬

```python

import heapq

class PriorityQueue:

    def __init__(self) :
        self.data = []

    def push (self, value):
        heapq.heappush(self.data, value)
        
    def pop (self) :
        if len(self.data) > 0 :
            heapq.heappop(self.data)
    
    def top(self) :
        if len(self.data) == 0:
            return -1
        else :
            return self.data[0]

def heapSort(items) :
    '''
    items에 있는 원소를 heap sort하여 리스트로 반환하는 함수를 작성하세요.

    단, 이전에 작성하였던 priorityQueue를 이용하세요.
    '''

    result = []
    
    pq = PriorityQueue()
    
    for item in items :
        pq.push(item)
    
    for i in range(len(items)) :
        result.append(pq.top())
        pq.pop()

    return result

```

#  05. 예제
## 05-1) 힘의 최솟값

```python

import heapq
def getMinForce(weights) :
    '''
    n개의 점토를 하나로 합치기 위해 필요한 힘의 합의 최솟값을 반환하는 함수를 작성하세요.
    '''
    pq = []
    result = 0
    
    #heapq.heappush()
    for w in weights :
        heapq.heappush(pq, w)
    while len(pq) > 1:
        x = heapq.heappop(pq)
        y = heapq.heappop(pq)
        
        temp = x+y
        result += temp
        
        heapq.heappush(pq, temp)
        

    return result
    
```

<br>

## 05-2) 중간값 찾기

```python

import heapq
def find_mid(nums) :
    '''
    n개의 정수들을 담고 있는 배열 nums가 매개변수로 주어집니다.
    nums의 각 정수들이 차례대로 주어질 때, 매 순간마다 
    "지금까지 입력된 수 중에서 중간값"을 리스트로 저장하여 반환하세요.
    '''
    
    n = len(nums)
    result = []
    l = [] # 최대 힙
    r = [] # 최소 힙
    
    for i in range(n) :
        x = nums[i]
        if not l or not r : #l 또는 r이 비어있는 경우
            heapq.heappush(l, -x)
        else :
            if x >= -l[0] :
                heapq.heappush(r, x) #루트노드 비교
            else :
                heapq.heappush(l, -x)
        
        # 두 힙 크기 조정
        # l과 r이 갖고있는 자료의 개수는 같아야하며
        #총 자료의 개수가 홀수면 l이 하나 더 많이 들고있게 해야함.
        
        while not (len(l) == len(r) or len(l) == len(r)+1) :
            #l이 들고있는 개수가 r의 개수보다 2개 이상이다.
            if len(l) > len(r):
                #l에서 값을 뽑아서 r에 넣는다.
                heapq.heappush(r, -heapq.heappop(l))
            # r이 l보다 많이 가진 경우
            else :
                # r에서 값을 뽑아서 l에 넣는다.
                heapq.heappush(l, -heapq.heappop(r))
                
        result.append(-l[0])

    
    return result
    
```

<br><br>
          
