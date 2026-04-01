Sleep Sort
How It Works
 Instead of sitting there comparing numbers against each other like every other sorting algorithm does, this one just makes each number wait.
 An example is like  you are a teacher and you tell every student in the class to go stand outside, then come back in after waiting a number of minutes equal to their exam score. The student who scored 2 waits 2 minutes. The student who scored 9 waits 9 minutes. Obviously the student with the lowest score walks back in first, then the next lowest, and so on. By the time everyone has returned, they have naturally lined themselves up in order without anyone ever comparing scores directly. That is exactly what sleep sort does, except the students are threads and the waiting is done in milliseconds.
In the code each number gets its own thread. That thread immediately goes to sleep for a duration equal to its value multiplied by a time unit (10ms in my program). When the thread wakes up it adds its number to the output list. Since smaller numbers sleep less, they wake up first and get added first. The result comes out sorted.
For descending order specifically,  the threads still sleep the same way but when each one wakes and tries to insert its value, the insertion logic checks whether the value belongs before or after what is already in the list, placing larger values toward the front. So the list fills from largest to smallest as threads wake.
________________________________________
Step by Step Example
I will use the list [3, 7, 1, 5, 2] and sort it in descending order.
The time unit is 10ms, so each number sleeps for its value times 10.
First, five threads are created at the same time, one for each number:
•	Thread carrying 3 goes to sleep for 30ms
•	Thread carrying 7 goes to sleep for 70ms
•	Thread carrying 1 goes to sleep for 10ms
•	Thread carrying 5 goes to sleep for 50ms
•	Thread carrying 2 goes to sleep for 20ms
All five are sleeping at the same time. Now we just watch them wake up:
10ms  → value 1 wakes up  → output list: [1]
20ms  → value 2 wakes up  → output list: [2, 1]
30ms  → value 3 wakes up  → output list: [3, 2, 1]
50ms  → value 5 wakes up  → output list: [5, 3, 2, 1]
70ms  → value 7 wakes up  → output list: [7, 5, 3, 2, 1]
By 70ms all threads are done and the list is fully sorted. The main program was just sitting there waiting for all of them to finish, and once they do it prints the result.
Final output → [7, 5, 3, 2, 1]
________________________________________
Time Complexity
This is where sleep sort gets interesting because it does not behave like a normal algorithm.
Best Case — O(max V)
The best situation is when all the values in your list are small. Since all threads sleep at the same time (not one after the other), the total time the program takes is just however long the biggest number sleeps. If your largest number is 5, the whole sort finishes in about 50ms regardless of whether you have 10 numbers or 1000 numbers. The length of the list barely matters here.
Average Case — O(max V)
For a typical random list the same logic applies. Threads always overlap so you are always just waiting on the biggest sleeper. The average case is the same as the best case in terms of wall time.
Worst Case — O(max V + n)
Things slow down a little when you have a very large maximum value or when many threads are all trying to write to the output list at exactly the same moment. The mutex forces them to queue up one at a time, so with many threads arriving simultaneously there is some extra waiting on top of the sleep time. That extra part grows with n, giving us O(max V + n).
For the comparison and swap operations specifically:
When a thread wakes up and inserts its value, it does a small scan to find the right position. Most of the time this is instant because the value just goes to the end. But if there are lots of duplicate values all waking at the same time, each one has to scan through everything already in the list. In the absolute worst case where every number is the same, this becomes O(n²). For normal random data it stays at O(n).
________________________________________
Space Complexity
O(n) — and it is a real problem in practice.
The program needs two things for each element. First, a slot in the output array to store the result. Second, and more importantly, a whole thread. Every thread the OS creates comes with its own stack memory, which on Windows is about 8KB per thread by default.
So if you feed this program a list of 10,000 numbers, it will try to spawn 10,000 threads simultaneously. That is roughly 80MB of stack memory just for the threads, before you even count the actual data. Most operating systems have a hard limit on how many threads can exist at once, so for large inputs the program would simply crash or refuse to create more threads.
This is the main reason sleep sort stays a curiosity and never shows up in real software. It is not that the time complexity is bad — it is that the memory cost and thread limit make it fall apart the moment your list gets large.
________________________________________
Running It on Different List Sizes
I ran the program on several lists of different sizes to see how it actually behaves. All values were random positive integers between 1 and 20, time unit set to 10ms.
List Size	Comparisons	Swaps	Time Taken
5	5	0	~70ms
10	11	1	~90ms
20	21	2	~90ms
50	52	4	~200ms
100	103	6	~200ms
A few things I noticed from these results:
The wall time barely changes as the list gets longer. Going from 5 elements to 100 elements only added about 130ms of total time, and most of that is because larger lists are more likely to contain a value closer to 20 (the max), not because having more elements slows things down.
Comparisons grow almost perfectly in line with n. For 5 elements there were 5 comparisons, for 100 elements there were 103. This matches the O(n) average case I described above.
Swaps stayed very low throughout because random data rarely produces the kind of tie situation that forces re-ordering on insertion.
The big takeaway is that if I changed the values from ranging between 1 and 20 to ranging between 1 and 10000, the sort time would jump to around 100 seconds even for a list of just 3 numbers. That is the real quirk of this algorithm — value size matters far more than list size.

