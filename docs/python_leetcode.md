# 1. 力扣题
## 1.1 两数之和

* 题目  
  给定一个整数数组 nums 和一个整数目标值 target，请你在该数组中找出和为目标值 target 的那两个整数，并返回它们的数组下标。你可以假设每种输入只会对应一个答案，并且你不能使用两次相同的元素。你可以按任意顺序返回答案。

  * 示例 1：  
      输入：nums = [2,7,11,15], target = 9  
      输出：[0,1]  
      解释：因为 nums[0] + nums[1] == 9 ，返回 [0, 1] 。  

  * 示例 2：  
      输入：nums = [3,2,4], target = 6  
      输出：[1,2]  

  * 示例 3：  
      输入：nums = [3,3], target = 6  
      输出：[0,1]  

  * 提示：  
    ```
    2 <= nums.length <= 104
    -109 <= nums[i] <= 109
    -109 <= target <= 109
    ```
    只会存在一个有效答案。  
进阶：你可以想出一个时间复杂度小于 O(n2) 的算法吗？  

* 解题    

  * 思路 1：   
    最容易想到的方法是枚举数组中的每一个数 x，寻找数组中是否存在 target - x。
    当我们使用遍历整个数组的方式寻找 target - x 时，需要注意到每一个位于 x 之前的元素都已经和 x 匹配过，因此不需要再进行匹配。而每一个元素不能被使用两次，所以我们只需要在 x 后面的元素中寻找 target - x。
    
    **Python 实现：**  
  
    ```python
    class Solution(object):
        def twoSum(self, nums: List[int], target: int) -> List[int]:
            for i in range(len(nums)):
                for j in range(i+1, len(nums)):
                    if nums[i] + nums[j] == target:
                        return [i,j]
            return []
    ```
    执行时间：1875 ms、内存消耗：17.86 MB、时间复杂度：O(n2)、空间复杂度：O(1)  
    
  * 思路 2（思路一延伸）：由于题目保证仅存在一个答案，故遍历过程中只要存在一个解，即可返回结果，数组 [2, 7, 11, 15]，目标为 9，伪代码：

    ```
    Round 1：先取元素 2，根据 9-2=7，下一步寻找7是否在剩下的元素中，若在则返回下标，此时答案为 `[0, 1]`
    Round 2：先取元素 7，根据 9-7=2，下一步寻找2是否在剩下的元素中，若在则返回下标，此时答案为 `[1, 0]`
    ······
    Round 4：先取元素 11，9-11=-2，-2 不在数组 `nums` 中，故排除此种情况
    ```
    
    **Python 实现：**  
  
    ```python
    class Solution(object):
        def twoSum(self, nums: List[int], target: int) -> List[int]:
            # 遍历列表
            for i in range(len(nums)):
                # 计算需要找到的下一个目标数字
                res = target - nums[i]
                # 遍历剩下的元素，查找是否存在该数字
                if res in nums[i+1:]:
                    # 若存在，返回答案。这里由于是两数之和，可采用.index()方法
                    # 获得目标元素在nums[i+1:]这个子数组中的索引后，还需加上i+1才是该元素在nums中的索引
                    return [i, nums[i+1:].index(res)+i+1]
            return []
    ```
    执行时间：335 ms、内存消耗：17.88 MB、时间复杂度：O(n2)、空间复杂度：O(1)
  
  * 思路 3：哈希表    
    **思路及算法**  
    注意到思路 1 的时间复杂度较高的原因是寻找 target - x 的时间复杂度过高。因此，我们需要一种更优秀的方法，能够快速寻找数组中是否存在目标元素。如果存在，我们需要找出它的索引。 使用哈希表，可以将寻找 target - x 的时间复杂度降低到从 O(N) 降低到 O(1)。 这样我们创建一个哈希表，对于每一个 x，我们首先查询哈希表中是否存在 target - x，然后将 x 插入到哈希表中，即可保证不会让 x 和自己匹配。

    **Python 实现：**  
  
    ```python
    class Solution:
        def twoSum(self, nums: List[int], target: int) -> List[int]:
            hashtable = dict()
            for i, num in enumerate(nums):
                if target - num in hashtable:
                    return [hashtable[target - num], i]
                hashtable[nums[i]] = i
            return []
    ```
    执行时间：3 ms、内存消耗：18.25 MB、时间复杂度：O(N)、空间复杂度：O(N)
    