本实验要求我们实现一个流重组器，负责将乱序的string类子串配合其序号进行重新排序后，写入我们再Lab0中实现的ByteStream。

这里就不讲探索过程了，直接写明最终结论：
- 每个序号Index都对应一个char型字符，我们重新排序实际上就是将所有字符按序号排好
- 函数的输入是string类和第一个字符的序号，我们可以将其拆散后使用
- 我们必须保证写入ByteStream的都是排好序的字符，因而我们需要有一个缓冲来记录提前到达的字符
- 由于实验情况中可能出现子串重叠，我们使用map来保证随机读取，从而减少时间复杂度
- 已经写入的字符很显然是不能再写入的，因而我们要对子串进行去重处理，具体实现就是保存一个当前应该写入的字符序号，以此来去重即可

最终的实现如下：
![[Pasted image 20230916160235.png]]