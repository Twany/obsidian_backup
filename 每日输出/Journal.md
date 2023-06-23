## 6.10
### 算法
- [x] [179. 最大数](https://leetcode.cn/problems/largest-number)
	- 自定义排序：(a+b).compareTo(b+a)
	- 使用快排：int x = 12; int y = 345 x 拼接 y = 12345 = 12 * 1000 + 345 = x * 1000 + y; y 拼接 x = 34512 = 345 * 100 + 12 = y * 100 + x; 上面的1000是哪里来的？因为y是3位数。上面的100是哪里来的？因为x是2位数；
- [x] [153. 寻找旋转排序数组中的最小值](https://leetcode.cn/problems/find-minimum-in-rotated-sorted-array)
	- 三种情况，边界问题
- [x] [24. 两两交换链表中的节点](https://leetcode.cn/problems/swap-nodes-in-pairs)
	- 简单递归
- [x] 每日一题：[1170. 比较字符串最小字母出现频次]([1170. 比较字符串最小字母出现频次 - 力扣（Leetcode）](https://leetcode.cn/problems/compare-strings-by-frequency-of-the-smallest-character/)
	- 后缀和

## 6.23
- [ ] 每日一题：[2496. 数组中字符串的最大值 - 力扣（Leetcode）](https://leetcode.cn/problems/maximum-value-of-a-string-in-an-array/description/)
	- [ ] 对于每个字符串，检查是否全是数字，如果是则转整数，不是则计算长度
- [ ] [468. 验证IP地址 - 力扣（Leetcode）](https://leetcode.cn/problems/validate-ip-address/description/)
		- [ ] 验证ipv4：split成四段，然后检验长度是否大于0小于4，长度大于1开头是否不为0，最后检验是否都是数字且和小于255
			- [ ] 和递归拆分ip地址那道题很像
	- [ ] 验证ipv6：和v4差不多
- [ ] [138. 复制带随机指针的链表](https://leetcode.cn/problems/copy-list-with-random-pointer)
	- [ ] 需要注意的是从克隆map取node赋值next和random时，不是取的原本node，而是clone的：深拷贝要保证完全拷贝，包括属性
	- [ ] 迭代法 + 哈希法

