
# 数组

## 找数组中的重复元素(不修改数组)

- 题目描述

> 在一个长度为n的数组里的所有数字都在0到n-1的范围内。 数组中某些数字是重复的，但不知道有几个数字是重复的。也不知道每个数字重复几次。请找出数组中任意一个重复的数字。 例如，如果输入长度为7的数组{2,3,1,0,2,5,3}，那么对应的输出是第一个重复的数字2。

------

- 法一（能找出所有重复元素）

  思想：用哈希表存储访问过的值

  具体步骤：从头到尾扫描数组，每扫描到一个数字就判断哈希表是否含有这个数字。如果哈希表还没有这个数字就把它加入哈希表，否则的话就是找到了重复的数字了，返回它即可。

```java
 public boolean duplicate(int numbers[], int length, int[] duplication) {
     
        for (int i = 0; i < k.length; i++) {
            if (k[numbers[i]] == true) {
                duplication[0] = numbers[i];
                return true;
            }
            k[numbers[i]] = true;
        }
        return false;
    }
```

- 法二（不能找出全部）

  思想：利用折半查找

  步骤：把1-n的数字从中间m分为两部分，前面一半1-m，后一半为m+1-n。
  如果1-m的数字数目超过m，那么这一半的区间里一定包含重复的数字；否则，另一半m+1-n的区间里一定包含重复的数字


```java
public boolean duplicate(int numbers[], int length, int[] duplication) {
        if (null == numbers || length <= 0) return false;
        for (int i = 0; i < numbers.length; i++)
            if (numbers[i] < 0 || (numbers[i] > length - 1))
                return false;

        int l = 0, r = length - 1;
        int count = 1;
        while (l < r) {
            int mid = (l + r) >>> 1;
            count = countRange(numbers, length, l, r);
            if (count > (mid - l + 1)) r = mid;
            else l = mid + 1;
        }
        if (count > 1) {
            duplication[0] = numbers[l];
            return true;
        }
        return false;
    }
```

## 找数组中的重复元素(可修改数组)

- 思想

  从头到尾一次扫描这个数组的每个数字，当扫描到下标为i的数字，先判断这个数字m是不是i,如果是，则继续扫描下一个数字，如果不是，那就把它跟第m个数字比较，如果相等，就找到重复的数字，如果不等就把第i个数跟第M个数交换，把M放到属于他的位置。接下来重复这个比较交换的操作直到找到重复数字。

```java
public boolean duplicate(int numbers[],int length,int [] duplication) {
        if(null==numbers || length<=0) return false;
        for(int i=0;i<numbers.length;i++)
            if( numbers[i]<0 || (numbers[i]>length-1) )
                return false;
        
        for(int i=0;i<numbers.length;i++){
            while( i!=numbers[i] ){//最多只要交换2次,为什么是2次?比如0,3,1,2
                if(numbers[i]==numbers[numbers[i]]){
                    duplication[0]=numbers[i];
                    return true;
                }
                int temp=numbers[i];
                numbers[i]=numbers[temp];
                numbers[temp]=temp;
            }    
        }
        return false;
    }
```

## 排序数组找某一重复元素下限

```java
int FindLowBoundary(int[] a, int target) {
    if (a == null || a.length == 0) return -1;

    int l = 0, r = a.length - 1;
    while (l < r) {
        int mid = (l + r) >>> 1;
        if (a[mid] < target) l = mid + 1;
        else                 r = mid;
    }
    if(target==a[l])  return l;
    else              return -1;
}
```

## 排序数组找某一重复元素上限

```java
int FindHighBoundary(int[] a, int target) {
    if (a == null || a.length == 0) return -1;

    int l = 0, r = a.length - 1;
    while (l < r) {
        int mid = (l + r+1) >>> 1;
        if (a[mid] > target) r = mid - 1;
        else                 l = mid;
    }
    if(target==a[l]) return l;
    else             return -1;
}
```

## leetcode81:搜索旋转数组(有重复数字)

- 思想

此题有三种情况

情况一. 2 3 4 5 6 7 1，即是左边有序的情况

情况二. 6 7 1 2 3 4 5，即是右边有序的情况

情况三.1 0 1 1 1和1 1 1 01这种，没办法分清楚是左边有序还是右边有序，只需左+1就可以

```java
public boolean search(int[] nums, int target) {
        int len = nums.length;
        if (len == 0) return false;
        
        int left = 0;
        int right = len - 1;
        while (left < right) {
            int mid = (left + right) >>> 1;
            if (nums[mid] > nums[left]) { // 落在前有序数组里
                 if (nums[left] <= target && target <= nums[mid])  right = mid;
                 else 											  left = mid + 1;
            } else if (nums[mid] < nums[left]) {
                if (nums[mid] < target && target <= nums[right])  left = mid + 1;
                else 										      right = mid;
            } else {					// 要排除掉左边界之前，先看一看左边界可以不可以排除
                if (nums[left] == target)  						  return true;
                else 											  left = left + 1;   
            }
        }
        return nums[left] == target;
    }
```

## 旋转数组的最小数字

- 思路

思路基本同上题一样，唯一一点不同的是，有可能会导致非递减数列，所以要直接返回

```java
public int minNumberInRotateArray(int [] array) {
        if(null==array || array.length==0) return 0;
        
        int l=0;
        int r=array.length-1;
        while(l<r){
             if (array[l] < array[r]) return array[l];
             int mid = (l + r) >>> 1;
             if (array[mid] > array[l])         l=mid+1;
             else if (array[mid] < array[l])    r=mid;
             else                               l++;
        }
        return array[l];
    }
```

## leetcode66:加一

> 给定一个由整数组成的非空数组所表示的非负整数，在该数的基础上加一。
>
> 最高位数字存放在数组的首位， 数组中每个元素只存储单个数字。
>
> 你可以假设除了整数 0 之外，这个整数不会以零开头。
>
> 示例 1:
>
> 输入: [1,2,3]
> 输出: [1,2,4]
> 解释: 输入数组表示数字 123。

------

```java
 public int[] plusOne(int[] digits) {
        for (int i = digits.length - 1; i >= 0; i--) {
            digits[i]++;
            digits[i] = digits[i] % 10;
            if (digits[i] != 0) return digits;
        }
        digits = new int[digits.length + 1];
        digits[0] = 1;
        return digits;
    }
```

## leetcode15:三数之和

> 给定一个包含 n 个整数的数组 nums，判断 nums 中是否存在三个元素 a，b，c ，使得 a + b + c = 0 ？找出所有满足条件且不重复的三元组。
>
> 注意：答案中不可以包含重复的三元组。
>
> 例如, 给定数组 nums = [-1, 0, 1, 2, -1, -4]，
>
> 满足要求的三元组集合为：
> [
>   [-1, 0, 1],
>   [-1, -1, 2]
> ]

------

- 思路


1.首先对数组进行排序，排序后固定一个数nums[i]，再使用左右指针指向 nums[i]后面的两端，数字分别为 nums[L] 和nums[R]，计算三个数的和 sum 判断是否满足为 00，满足则添加进结果集
2.如果 nums[i]大于 00，则三数之和必然无法等于 00，结束循环
3.如果 nums[i] == nums[i−1]，则说明该数字重复，会导致结果重复，所以应该跳过
当sum == 0 时，nums[L] == nums[L+1] 则会导致结果重复，应该跳过，L++
当sum = 0 时，nums[R] == nums[R−1] 则会导致结果重复，应该跳过，R--

```java
  List<List<Integer>> ans = new ArrayList();
        int len = nums.length;
        if(nums == null || len < 3) return ans;
        Arrays.sort(nums); // 排序
        for (int i = 0; i < len ; i++) {
            if(nums[i] > 0) break; // 如果当前数字大于0，则三数之和一定大于0，所以结束循环
            if(i > 0 && nums[i] == nums[i-1]) continue; // 去重
            int L = i+1;
            int R = len-1;
            while(L < R){
                int sum = nums[i] + nums[L] + nums[R];
                if(sum == 0){
                    ans.add(Arrays.asList(nums[i],nums[L],nums[R]));
                    while (L<R && nums[L] == nums[L+1]) L++; // 去重
                    while (L<R && nums[R] == nums[R-1]) R--; // 去重
                    L++;
                    R--;
                }
                else if (sum < 0) L++;
                else if (sum > 0) R--;
            }
        }        
        return ans;
    }
```

# 链表

## 链表转置

- 法一：头插法


- 法二：递归法


- 法三：三指针法，关键是要记录左右节点


```java
void reverse(ListNode head) {
	if (head == null) return;
	ListNode prev = null;
	ListNode cur = head;
	ListNode next = head;
	while(cur!=null){
		next=cur.next;
		cur.next=prev;
		prev=cur;
		cur=next;
	}
}
```

## 链表中环的入口结点·

- 思想：快慢指针


```java
ListNode EntryNodeOfLoop(ListNode pHead){
	if(pHead==null || pHead.next==null || pHead.next.next==null) return null;
	ListNode fast=pHead.next.next;
	ListNode low=pHead.next;
	while( fast!=low  ){
		if(fast.next!=null && fast.next.next!=null){
			fast=fast.next.next;
			low=low.next;
		}
		else return null;
	}
	ListNode p= pHead;
	while(p!=fast){
		p=p.next;
		fast=fast.next;
	}
	return p;
}
```

## 从尾到头输出链表

- 法一：递归；


- 法二：用栈；


- 法三：从头到尾放入List后再将List转置。


```java
//法一：递归代码
ArrayList<Integer> list =new ArrayList<>();

private ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
    GetListFromTailToHead(listNode);
    return list;
}

private void GetListFromTailToHead(ListNode p){
    if(p==null) return;
    GetListFromTailToHead(p.next,);
    list.add(p.val);
}
```

## 链表中倒数第K个结点

- 思想：快指针先走k-1步，然后慢指针


```java
public ListNode FindKthToTail(ListNode head,int k) {
        if(head==null || k<=0) return null;
        ListNode low=head,fast=head;
        for(int i=0;i<k-1;i++){
            if(fast.next==null) return null;
            fast=fast.next;
        }
        while(fast.next!=null) {
            fast=fast.next;
            low=low.next;
        } 
        return low;
    }
```

## 求两链表的交叉点

- 思想：快慢指针，先让




## leetcode2:两数相加

> 给出两个非空 的链表用来表示两个非负的整数。其中，它们各自的位数是按照逆序的方式存储的，并且它们的每个节点只能存储一位数字。
>
> 如果，我们将这两个数相加起来，则会返回一个新的链表来表示它们的和。
>
> 您可以假设除了数字 0 之外，这两个数都不会以 0 开头。
>
> 示例：
>
> 输入：(2 -> 4 -> 3) + (5 -> 6 -> 4)
> 输出：7 -> 0 -> 8
> 原因：342 + 465 = 807

------

- 思路


1.把两个链表当做相同长度遍历，不够的补0

2.需要考虑进位问题

3.遍历完后，需要考虑会不会产生新的一位

```java
public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode pre = new ListNode(0);
        ListNode cur = pre;
        int carry = 0;
        while(l1 != null || l2 != null) {
            int x = l1 == null ? 0 : l1.val;
            int y = l2 == null ? 0 : l2.val;
            int sum = x + y + carry;
            
            carry = sum / 10;
            sum = sum % 10;
            cur.next = new ListNode(sum);

            cur = cur.next;
            if(l1 != null) l1 = l1.next;
            if(l2 != null) l2 = l2.next;
        }
        if(carry == 1) {
            cur.next = new ListNode(carry);
        }
        return pre.next;
    }
```

# 字符串

## leetcode31：下一排列

> 实现获取下一个排列的函数，算法需要将给定数字序列重新排列成字典序中下一个更大的排列。
>
> 如果不存在下一个更大的排列，则将数字重新排列成最小的排列（即升序排列）。
>
> 必须原地修改，只允许使用额外常数空间。
>
> 以下是一些例子，输入位于左侧列，其相应输出位于右侧列。
> 1,2,3 → 1,3,2
> 3,2,1 → 1,2,3
> 1,1,5 → 1,5,1
>

------

- 思路

从右到左找第一个非递增的元素i，然后从该元素从右找比他大的最小元素j，交换,最后转置i+1-n的所有元素。

------

```java
 public void nextPermutation(int[] nums) {
        int i = nums.length - 2;
        while (i >= 0 && nums[i + 1] <= nums[i]) i--;
        
        if (i >= 0) {
            int j = nums.length - 1;
            while (j >= 0 && nums[j] <= nums[i]) {
                j--;
            }
            swap(nums, i, j);
        }
        reverse(nums, i + 1);
    }

    private void reverse(int[] nums, int start) {
        int i = start, j = nums.length - 1;
        while (i < j) {
            swap(nums, i, j);
            i++;
            j--;
        }
    }

    private void swap(int[] nums, int i, int j) {
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
```

## 字符串排列(数字排列)

- 法一


找出最小的排列（ Collections.sort(list) ），然后用找下一排列的方法直至找不到

- 法二
  步骤一：固定第一个字符，递归取得首位后面的各种字符串组合；
  步骤二：再把第一个字符与后面每一个字符交换，并同样递归获得首位后面的字符串组合； 
  递归的出口，就是只剩一个字符的时候
  递归的循环过程，就是从每个子串的第二个字符开始依次与第一个字符交换，然后继续处理子串。


```java
题目描述
输入一个字符串,按字典序打印出该字符串中字符的所有排列。例如输入字符串abc,则打印出由字符a,b,c所能排列出来的所有字符串abc,acb,bac,bca,cab和cba。

输入描述:输入一个字符串,长度不超过9(可能有字符重复),字符只包括大小写字母

    ArrayList<String> list = new ArrayList<>();
    
    private void help(char[] chars,int begin,int end){
        if(begin==end) {
            list.add(String.valueOf(chars));
        }else{
            Set<Character> set= new HashSet<>();
            for(int i=begin;i<=end;i++){
                if(  !set.contains(chars[i])  ){
                     set.add(chars[i]);
                     swap(chars,i,begin);
                     help(chars,begin+1,end);
                     swap(chars,i,begin);
                }
                   
            }
        }
    }

```

## 字符串的组合

- 思想

先从头扫描字符串的第一个字符。针对第一个字符，我们有两种选择：第一是把这个字符放到组合中去，接下来我们需要在剩下的n-1个字符中选取m-1个字符；第二是不把这个字符放到组合中去，接下来我们需要在剩下的n-1个字符中选择m个字符。

```java
   ArrayList<String> list = new ArrayList<>();
    HashSet<String> set = new HashSet<>();

    Solution(){
        String s="ABC";
        char[] c=s.toCharArray();
        StringBuffer sb=new StringBuffer();
        for(int i=1;i<=c.length;i++) {
            help(c,0,i,sb);
        }
    }
    
    private void help(char[] str, int start, int len, StringBuffer sb) {
        if (len == 0) {
            if (!set.contains(sb.toString()))
                list.add(sb.toString());
            return;
        }
        if (start == str.length) return;
        sb.append(str[start]);
        help(str, start + 1, len - 1, sb);
        sb.deleteCharAt(sb.length() - 1);//防止脏数据
        help(str, start + 1, len, sb);
    }
```

# 回溯法

## 1.N皇后问题




