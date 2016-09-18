# 一、定义 #

1. 消息队列在Android中指的是MessageQueue。
2. MessageQueue是一个通过单链表的数据结构来维护的消息列表。
2. MessageQueue主要包含两个操作：插入和读取。
3. 插入和读取分别对应enqueueMessage和next方法。
3. MessageQueue之所以用单链表来实现，是因为MessageQueue中有频繁的插入和删除操作，而这些恰恰是单链表这种数据结构所擅长的。

# 二、原理 #

1. enqueueMessage的操作主要就是单链表的插入，而next则是一个无限循环的方法。

1. 如果在消息队列中没有消息，那么next方法会一直阻塞在这里。当有新消息来的时候，next方法会返回这条消息并将其从单链表中移除。
