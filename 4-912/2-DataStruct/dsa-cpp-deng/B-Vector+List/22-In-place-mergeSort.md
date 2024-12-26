---
publish: "true"
---
## Intro

Implement [Merge Sort](https://www.geeksforgeeks.org/merge-sort/) i.e. standard implementation keeping the sorting algorithm as in-place.   

In-place means it does not occupy extra memory for merge operation as in the standard case.

## Approach 1: two pointers

- Maintain two pointers that point to the start of the segments which have to be merged.
- Compare the elements at which the pointers are present.
- If _element1 < element2_ then _element1_ is at right position, simply increase _pointer1_.
- Else shift all the elements between _element1_ and _element2(including element1 but excluding element2)_ right by 1 and then place the element2 in the previous place _(i.e. before shifting right)_ of _element1_. Increment all the pointers by _1_.

### Implementation

Below is the implementation of the above approach:

```Cpp
// C++ program in-place Merge Sort
#include <iostream>
using namespace std;

// Merges two subarrays of arr[].
// First subarray is arr[l..m]
// Second subarray is arr[m+1..r]
// Inplace Implementation
void merge(int arr[], int start, int mid, int end)
{
	int start2 = mid + 1;

	// If the direct merge is already sorted
	if (arr[mid] <= arr[start2]) {
		return;
	}

	// Two pointers to maintain start
	// of both arrays to merge
	while (start <= mid && start2 <= end) {

		// If element 1 is in right place
		if (arr[start] <= arr[start2]) {
			start++;
		}
		else {
			int value = arr[start2];
			int index = start2;

			// Shift all the elements between element 1
			// element 2, right by 1.
			while (index != start) {
				arr[index] = arr[index - 1];
				index--;
			}
			arr[start] = value;

			// Update all the pointers
			start++;
			mid++;
			start2++;
		}
	}
}

/* l is for left index and r is right index of the
sub-array of arr to be sorted */
void mergeSort(int arr[], int l, int r)
{
	if (l < r) {

		// Same as (l + r) / 2, but avoids overflow
		// for large l and r
		int m = l + (r - l) / 2;

		// Sort first and second halves
		mergeSort(arr, l, m);
		mergeSort(arr, m + 1, r);

		merge(arr, l, m, r);
	}
}

/* UTILITY FUNCTIONS */
/* Function to print an array */
void printArray(int A[], int size)
{
	int i;
	for (i = 0; i < size; i++)
		cout <<" "<< A[i];
	cout <<"\n";
}

/* Driver program to test above functions */
int main()
{
	int arr[] = { 12, 11, 13, 5, 6, 7 };
	int arr_size = sizeof(arr) / sizeof(arr[0]);

	mergeSort(arr, 0, arr_size - 1);

	printArray(arr, arr_size);
	return 0;
}

// This code is contributed by shivanisinghss2110

```

```c
// C++ program in-place Merge Sort
#include <stdio.h>

// Merges two subarrays of arr[].
// First subarray is arr[l..m]
// Second subarray is arr[m+1..r]
// Inplace Implementation
void merge(int arr[], int start, int mid, int end)
{
	int start2 = mid + 1;

	// If the direct merge is already sorted
	if (arr[mid] <= arr[start2]) {
		return;
	}

	// Two pointers to maintain start
	// of both arrays to merge
	while (start <= mid && start2 <= end) {

		// If element 1 is in right place
		if (arr[start] <= arr[start2]) {
			start++;
		}
		else {
			int value = arr[start2];
			int index = start2;

			// Shift all the elements between element 1
			// element 2, right by 1.
			while (index != start) {
				arr[index] = arr[index - 1];
				index--;
			}
			arr[start] = value;

			// Update all the pointers
			start++;
			mid++;
			start2++;
		}
	}
}

/* l is for left index and r is right index of the
sub-array of arr to be sorted */
void mergeSort(int arr[], int l, int r)
{
	if (l < r) {

		// Same as (l + r) / 2, but avoids overflow
		// for large l and r
		int m = l + (r - l) / 2;

		// Sort first and second halves
		mergeSort(arr, l, m);
		mergeSort(arr, m + 1, r);

		merge(arr, l, m, r);
	}
}

/* UTILITY FUNCTIONS */
/* Function to print an array */
void printArray(int A[], int size)
{
	int i;
	for (i = 0; i < size; i++)
		printf("%d ", A[i]);
	printf("\n");
}

/* Driver program to test above functions */
int main()
{
	int arr[] = { 12, 11, 13, 5, 6, 7 };
	int arr_size = sizeof(arr) / sizeof(arr[0]);

	mergeSort(arr, 0, arr_size - 1);

	printArray(arr, arr_size);
	return 0;
}

```

```python
# Python program in-place Merge Sort

# Merges two subarrays of arr.
# First subarray is arr[l..m]
# Second subarray is arr[m+1..r]
# Inplace Implementation


def merge(arr, start, mid, end):
	start2 = mid + 1

	# If the direct merge is already sorted
	if (arr[mid] <= arr[start2]):
		return

	# Two pointers to maintain start
	# of both arrays to merge
	while (start <= mid and start2 <= end):

		# If element 1 is in right place
		if (arr[start] <= arr[start2]):
			start += 1
		else:
			value = arr[start2]
			index = start2

			# Shift all the elements between element 1
			# element 2, right by 1.
			while (index != start):
				arr[index] = arr[index - 1]
				index -= 1

			arr[start] = value

			# Update all the pointers
			start += 1
			mid += 1
			start2 += 1


'''
* l is for left index and r is right index of
the sub-array of arr to be sorted
'''


def mergeSort(arr, l, r):
	if (l < r):

		# Same as (l + r) / 2, but avoids overflow
		# for large l and r
		m = l + (r - l) // 2

		# Sort first and second halves
		mergeSort(arr, l, m)
		mergeSort(arr, m + 1, r)

		merge(arr, l, m, r)


''' UTILITY FUNCTIONS '''
''' Function to print an array '''


def printArray(A, size):

	for i in range(size):
		print(A[i], end=" ")
	print()


''' Driver program to test above functions '''
if __name__ == '__main__':
	arr = [12, 11, 13, 5, 6, 7]
	arr_size = len(arr)

	mergeSort(arr, 0, arr_size - 1)
	printArray(arr, arr_size)

# This code is contributed by 29AjayKumar

```

### Complexity

Note: Time Complexity of above approach is O(n^2 * log(n)) because merge is O(n^2). Time complexity of standard merge sort is less, O(n log n).

## Approach 2: comparing elements farly

We start comparing elements that are far from each other rather than adjacent. Basically we are using *shell sorting* to merge two sorted arrays with O(1) extra space.

mergeSort(): 
- Calculate mid two split the array in two halves(left sub-array and right sub-array)
- Recursively call merge sort on left sub-array and right sub-array to sort them
- Call merge function to merge left sub-array and right sub-array

merge():
- For every pass, we calculate the gap and compare the elements towards the right of the gap.
- Initiate the gap with ceiling value of n/2 where n is the combined length of left and right sub-array.
- Every pass, the gap reduces to the ceiling value of gap/2.
- Take a pointer i to pass the array.
- Swap the ith and (i+gap)th elements if (i+gap)th element is smaller than(or greater than when sorting in decreasing order) ith element.
- Stop when (i+gap) reaches n.

```
Input: 10, 30, 14, 11, 16, 7, 28

Note: Assume left and right subarrays has been sorted so we are merging sorted subarrays [10, 14, 30] and [7, 11, 16, 28]

Start with

gap =  ceiling of n/2 = 7/2 = 4

[This gap is for whole merged array]

10, 14, 30, 7, 11, 16, 28

10, 14, 30, 7, 11, 16, 28

10, 14, 30, 7, 11, 16, 28

10, 14, 28, 7, 11, 16, 30

gap =  ceiling of 4/2 = 2

10, 14, 28, 7, 11, 16, 30

10, 14, 28, 7, 11, 16, 30

10, 7, 28, 14, 11, 16, 30

10, 7, 11, 14, 28, 16, 30

10, 7, 11, 14, 28, 16, 30

 

gap =  ceiling of 2/2 = 1

10, 7, 11, 14, 28, 16, 30

7, 10, 11, 14, 28, 16, 30

7, 10, 11, 14, 28, 16, 30

7, 10, 11, 14, 28, 16, 30

7, 10, 11, 14, 28, 16, 30

7, 10, 11, 14, 16, 28, 30

 

Output: 7, 10, 11, 14, 16, 28, 30
```

### Implementation

```cpp
// C++ program for the above approach
#include <bits/stdc++.h>
using namespace std;

// Calculating next gap
int nextGap(int gap)
{
	if (gap <= 1)
		return 0;
		
	return (int)ceil(gap / 2.0);
}

// Function for swapping
void swap(int nums[], int i, int j)
{
	int temp = nums[i];
	nums[i] = nums[j];
	nums[j] = temp;
}

// Merging the subarrays using shell sorting
// Time Complexity: O(nlog n)
// Space Complexity: O(1)
void inPlaceMerge(int nums[], int start,
							int end)
{
	int gap = end - start + 1;
	
	for(gap = nextGap(gap); 
		gap > 0; gap = nextGap(gap)) 
	{
		for(int i = start; i + gap <= end; i++) 
		{
			int j = i + gap;
			if (nums[i] > nums[j])
				swap(nums, i, j);
		}
	}
}

// merge sort makes log n recursive calls
// and each time calls merge()
// which takes nlog n steps
// Time Complexity: O(n*log n + 2((n/2)*log(n/2)) +
// 4((n/4)*log(n/4)) +.....+ 1)
// Time Complexity: O(logn*(n*log n))
// i.e. O(n*(logn)^2)
// Space Complexity: O(1)
void mergeSort(int nums[], int s, int e)
{
	if (s == e)
		return;

	// Calculating mid to slice the
	// array in two halves
	int mid = (s + e) / 2;

	// Recursive calls to sort left
	// and right subarrays
	mergeSort(nums, s, mid);
	mergeSort(nums, mid + 1, e);
	
	inPlaceMerge(nums, s, e);
}

// Driver Code
int main()
{
	int nums[] = { 12, 11, 13, 5, 6, 7 };
	int nums_size = sizeof(nums) / sizeof(nums[0]);
	
	mergeSort(nums, 0, nums_size-1);
	
	for(int i = 0; i < nums_size; i++)
	{
		cout << nums[i] << " ";
	}
	return 0;
}

// This code is contributed by adityapande88

```

```python
# Python3 program for the above approach
import math 

# Calculating next gap
def nextGap(gap):

	if gap <= 1:
		return 0
		
	return int(math.ceil(gap / 2))

# Function for swapping
def swap(nums, i, j):

	temp = nums[i]
	nums[i] = nums[j]
	nums[j] = temp

# Merging the subarrays using shell sorting
# Time Complexity: O(nlog n)
# Space Complexity: O(1)
def inPlaceMerge(nums,start, end):

	gap = end - start + 1
	gap = nextGap(gap)

	while gap > 0:
		i = start
		while (i + gap) <= end:
			j = i + gap
			
			if nums[i] > nums[j]:
				swap(nums, i, j)
				
			i += 1
		
		gap = nextGap(gap)
			
# merge sort makes log n recursive calls
# and each time calls merge()
# which takes nlog n steps
# Time Complexity: O(n*log n + 2((n/2)*log(n/2)) +
# 4((n/4)*log(n/4)) +.....+ 1)
# Time Complexity: O(logn*(n*log n))
# i.e. O(n*(logn)^2)
# Space Complexity: O(1)
def mergeSort(nums, s, e):

	if s == e:
		return

	# Calculating mid to slice the
	# array in two halves
	mid = (s + e) // 2

	# Recursive calls to sort left
	# and right subarrays
	mergeSort(nums, s, mid)
	mergeSort(nums, mid + 1, e)
	
	inPlaceMerge(nums, s, e)

# UTILITY FUNCTIONS 
# Function to print an array 
def printArray(A, size):

	for i in range(size):
		print(A[i], end = " ")
		
	print()

# Driver Code
if __name__ == '__main__':
	
	arr = [ 12, 11, 13, 5, 6, 7 ]
	arr_size = len(arr)

	mergeSort(arr, 0, arr_size - 1)
	printArray(arr, arr_size)

# This code is contributed by adityapande88

```

### Complexity

**Time Complexity:** O(logn * nlogn)

**Note:** mergeSort method makes logn recursive calls and each time merge is called which takes nlogn time to merge 2 sorted sub-arrays.

## Approach 3: number convertion

Suppose we have a number A and we want to convert it to a number B and there is also a constraint that we can recover number A any time without using other variable.To achieve this we choose a number N which is greater  
than both numbers and add `B*N` in A.

so `A --> A+B*N`

To get number B out of `(A+B*N)`, we divide `(A+B*N)` by N:  `(A+B*N)/N = B`.

To get number A out of `(A+B*N)`, we take modulo with N: `(A+B*N)%N = A`.

In a short, by taking modulo, we get old number back and taking divide we new number.

**mergeSort():**

- Calculate mid two split the array into two halves(left sub-array and right sub-array)
- Recursively call merge sort on left sub-array and right sub-array to sort them
- Call merge function to merge left sub-array and right sub-array

**merge():**

- We first find the maximum element of both sub-array and increment it one to avoid collision of 0 and maximum element during modulo operation.
- The idea is to traverse both sub-arrays from starting simultaneously. One starts from l till m and another starts from m+1 till r. So, We will initialize 3 pointers say i, j, k.
- i will move from l till m; j will move from m+1 till r; k will move from l till r.
- Now update value a[k] by adding min(a[i],a[j])*maximum_element.
- Then also update those elements which are left in both sub-arrays.
- After updating all the elements divide all the elements by maximum_element so we get the updated array back.

### Implementation

```cpp
// C++ program in-place Merge Sort
#include <bits/stdc++.h>
using namespace std;

// Merges two subarrays of arr[].
// First subarray is arr[l..m]
// Second subarray is arr[m+1..r]
// Inplace Implementation
void mergeInPlace(int a[], int l, int m, int r)
{
	// increment the maximum_element by one to avoid
	// collision of 0 and maximum element of array in modulo
	// operation
	int mx = max(a[m], a[r]) + 1;

	int i = l, j = m + 1, k = l;
	while (i <= m && j <= r && k <= r) {

		// recover back original element to compare
		int e1 = a[i] % mx;
		int e2 = a[j] % mx;
		if (e1 <= e2) {
			a[k] += (e1 * mx);
			i++;
			k++;
		}
		else {
			a[k] += (e2 * mx);
			j++;
			k++;
		}
	}

	// process those elements which are left in the array
	while (i <= m) {
		int el = a[i] % mx;
		a[k] += (el * mx);
		i++;
		k++;
	}

	while (j <= r) {
		int el = a[j] % mx;
		a[k] += (el * mx);
		j++;
		k++;
	}

	// finally update elements by dividing with maximum
	// element
	for (int i = l; i <= r; i++)
		a[i] /= mx;
}

/* l is for left index and r is right index of the
sub-array of arr to be sorted */
void mergeSort(int arr[], int l, int r)
{
	if (l < r) {

		// Same as (l + r) / 2, but avoids overflow
		// for large l and r
		int m = l + (r - l) / 2;

		// Sort first and second halves
		mergeSort(arr, l, m);
		mergeSort(arr, m + 1, r);
		mergeInPlace(arr, l, m, r);
	}
}

// Driver Code
int main()
{
	int nums[] = { 12, 11, 13, 5, 6, 7 };
	int nums_size = sizeof(nums) / sizeof(nums[0]);

	mergeSort(nums, 0, nums_size - 1);

	for (int i = 0; i < nums_size; i++) {
		cout << nums[i] << " ";
	}
	return 0;
}

// This code is contributed by soham11806959

```

```python
def mergeInPlace(a, l, m, r):
	mx = max(a[m], a[r]) + 1

	i, j, k = l, m+1, l
	while i <= m and j <= r and k <= r:
		e1 = a[i] % mx
		e2 = a[j] % mx
		if e1 <= e2:
			a[k] += (e1 * mx)
			i += 1
			k += 1
		else:
			a[k] += (e2 * mx)
			j += 1
			k += 1

	while i <= m:
		el = a[i] % mx
		a[k] += (el * mx)
		i += 1
		k += 1

	while j <= r:
		el = a[j] % mx
		a[k] += (el * mx)
		j += 1
		k += 1

	for i in range(l, r+1):
		a[i] //= mx


def mergeSort(arr, l, r):
	if l < r:
		m = l + (r - l) // 2

		mergeSort(arr, l, m)
		mergeSort(arr, m + 1, r)
		mergeInPlace(arr, l, m, r)


nums = [12, 11, 13, 5, 6, 7]
nums_size = len(nums)
mergeSort(nums, 0, nums_size - 1)

for i in range(nums_size):
	print(nums[i], end=' ')

```

### Complexity

**Time Complexity:** O(n log n)  
**Note:**  Time Complexity of above approach is O(n^2) because merge is O(n). Time complexity of standard merge sort is  O(n log n).